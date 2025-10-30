using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數 =====
        private const int MaxOrders = 22;    // BIST_PT_ORD0..ORD21
        private const int TotalMask = 0x1F;  // BIST_PT_TOTAL[4:0] = (Total - 1)
        private const int OrdMask   = 0x3F;  // ORD 低6bit = pattern index (0..63)

        // ===== 狀態 =====
        private JObject _cfg;
        private bool _preferHw;     // true：以硬體真值優先
        private bool _isHydrating;  // 回填過程中的抑制旗標（避免連鎖 setter）

        // JSON 宣告的 min/max（會被夾到 1..22），用來建/擴清單
        private int _totalMin = 1;
        private int _totalMax = MaxOrders;

        // ===== 綁定集合 =====
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ===== 綁定屬性 =====
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        // 注意：Total 的邏輯很單純，只負責觸發 ResizeOrders；安全性在呼叫端保證
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        public ICommand ApplyCommand { get; }

        // ===== 建構子：先撐起安全清單，避免初載紅框 =====
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            // 先給 UI 一組穩定 ItemsSource
            TotalOptions.Clear();
            for (int v = 1; v <= MaxOrders; v++) TotalOptions.Add(v);

            // 初值 1（SelectedValue 會在清單內，不紅）
            Total = 1;
        }

        // ===== 對外 API：載入 JSON =====
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = _preferHw || preferHw;   // 一旦 true 就保留

            // 1) Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) PatternOptions 先建好（避免內層 Combo 沒 ItemsSource）
            EnsurePatternOptionsFromJson();

            // 3) 先建 TotalOptions，再設定 Total（是關鍵）
            var totalObj = autoRunControl["total"] as JObject;
            int jsonMin = totalObj != null && totalObj["min"] != null ? totalObj["min"].Value<int>() : 1;
            int jsonMax = totalObj != null && totalObj["max"] != null ? totalObj["max"].Value<int>() : MaxOrders;
            int jsonDef = totalObj != null && totalObj["default"] != null ? totalObj["default"].Value<int>() : MaxOrders;

            BuildTotalOptions(jsonMin, jsonMax); // 先把清單建好

            _isHydrating = true;
            try
            {
                if (_preferHw)
                {
                    // 硬體優先：Refresh 會自行確保清單覆蓋硬體值再設定 Total
                    RefreshFromHardware();
                }
                else
                {
                    // JSON path：先夾回範圍，再安全設定 Total
                    int def = Clamp(jsonDef, _totalMin, _totalMax);
                    SafeSetTotal(def);
                    HydrateOrdersFromJsonDefaultsForRange(0, def);
                }
            }
            finally
            {
                _isHydrating = false;
            }

            System.Diagnostics.Debug.WriteLine($"[AutoRun] LoadFrom ok. preferHw={_preferHw}, TotalOptions={TotalOptions.Count}, Total={Total}");
        }

        // ===== 寫入暫存器 =====
        public void Apply()
        {
            if (Total <= 0) return;

            int count = Clamp(Total, 1, MaxOrders);

            // 寫 TOTAL：0=1張
            byte totalRaw = (byte)((count - 1) & TotalMask);
            RegMap.Write8("BIST_PT_TOTAL", totalRaw);
            System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL raw=0x{totalRaw:X2} (UI={count})");

            // 寫 ORD0..ORD{count-1}
            for (int i = 0; i < count && i < Orders.Count; i++)
            {
                string reg = "BIST_PT_ORD" + i;
                byte val = (byte)(Orders[i].SelectedIndex & OrdMask);
                RegMap.Write8(reg, val);
                System.Diagnostics.Debug.WriteLine($"[AutoRun] Write {reg} = 0x{val:X2} ({LookupName(val)})");
            }

            _preferHw = true;
            RefreshFromHardware();
        }

        // ===== 回讀暫存器 =====
        public void RefreshFromHardware()
        {
            try
            {
                EnsurePatternOptionsFromJson();

                // 1) 讀 TOTAL → 轉 UI 值（1..22）
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int totalHw = Clamp(((int)totalRaw & TotalMask) + 1, 1, MaxOrders);

                // 2) 先確保清單吃得下硬體值，再設定 Total（解紅框關鍵）
                _isHydrating = true;
                try
                {
                    EnsureTotalOptionsCovers(totalHw);
                    SafeSetTotal(totalHw);
                    ResizeOrders(totalHw);
                }
                finally
                {
                    _isHydrating = false;
                }

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL raw=0x{totalRaw:X2} → UI={totalHw}");

                // 3) 逐張回讀 ORD
                for (int i = 0; i < totalHw; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte val = RegMap.Read8(reg);
                    int idx = val & OrdMask;

                    if (!OptionExists(idx))
                        idx = DefaultOptionIndex();

                    Orders[i].SelectedIndex = idx;
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Read {reg} = 0x{val:X2} → idx={idx} ({LookupName(idx)})");
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware done.\n");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ===== 內部：安全設定 Total（先確保清單，再設值）=====
        private void SafeSetTotal(int v)
        {
            v = Clamp(v, 1, MaxOrders);
            EnsureTotalOptionsCovers(v); // 先確保 ItemsSource 包住 v
            Total = v;                   // 再設定 SelectedValue
        }

        // ===== 內部：建 TotalOptions 清單（依 JSON 範圍，夾 1..22）=====
        private void BuildTotalOptions(int min, int max)
        {
            _totalMin = Clamp(min, 1, MaxOrders);
            _totalMax = Clamp(max, 1, MaxOrders);
            if (_totalMin > _totalMax) { var t = _totalMin; _totalMin = _totalMax; _totalMax = t; }

            TotalOptions.Clear();
            for (int v = _totalMin; v <= _totalMax; v++)
                TotalOptions.Add(v);
        }

        // ===== 內部：若硬體值超過 JSON 上限，就把清單擴到硬體值（最多 22）=====
        private void EnsureTotalOptionsCovers(int required)
        {
            int needMax = Clamp(required, 1, MaxOrders);

            if (TotalOptions.Count == 0)
            {
                for (int v = 1; v <= needMax; v++) TotalOptions.Add(v);
                _totalMin = 1; _totalMax = needMax;
                return;
            }

            int curMin = TotalOptions[0];
            int curMax = TotalOptions[TotalOptions.Count - 1];

            if (needMax > curMax)
            {
                for (int v = curMax + 1; v <= needMax; v++)
                    TotalOptions.Add(v);
                _totalMax = needMax;
            }

            // 理論上不會，但保險：若不是從 1 起算，重建 1.._totalMax
            if (curMin > 1)
            {
                TotalOptions.Clear();
                for (int v = 1; v <= _totalMax; v++)
                    TotalOptions.Add(v);
                _totalMin = 1;
            }
        }

        // ===== 內部：調整 slot 數（_preferHw=true 時，新增部分從硬體補值）=====
        private void ResizeOrders(int desired)
        {
            int need = Clamp(desired, 1, MaxOrders);
            int old = Orders.Count;

            // 擴
            while (Orders.Count < need)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            // 縮
            while (Orders.Count > need)
                Orders.RemoveAt(Orders.Count - 1);

            // 硬體優先：Total 變大時，只補新加的 slot（不動舊的）
            if (_preferHw && need > old)
            {
                for (int i = old; i < need; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    try
                    {
                        byte val = RegMap.Read8(reg);
                        int idx = val & OrdMask;
                        if (!OptionExists(idx)) idx = DefaultOptionIndex();
                        Orders[i].SelectedIndex = idx;

                        System.Diagnostics.Debug.WriteLine(
                            $"[AutoRun] Grow slot {reg} <- 0x{val:X2} (idx={idx} {LookupName(idx)})");
                    }
                    catch
                    {
                        Orders[i].SelectedIndex = DefaultOptionIndex();
                    }
                }
            }

            // 重編號（1-based）
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        // ===== JSON 預設填入（第一次載入用）=====
        private void HydrateOrdersFromJsonDefaultsForRange(int start, int count)
        {
            if (_cfg == null) return;

            var ordersObj = _cfg["orders"] as JObject;
            var defArray = ordersObj != null ? ordersObj["defaults"] as JArray : null;

            if (defArray == null || defArray.Count == 0)
            {
                // fallback：用 options 順序
                for (int i = 0; i < count && (i + start) < Orders.Count; i++)
                {
                    int idx = (i + start) < PatternOptions.Count ? PatternOptions[i + start].Index : DefaultOptionIndex();
                    Orders[i + start].SelectedIndex = idx;
                }
                return;
            }

            for (int i = 0; i < count && (i + start) < Orders.Count; i++)
            {
                int defIdx = defArray.Count > (i + start) ? defArray[i + start].Value<int>() : DefaultOptionIndex();
                if (!OptionExists(defIdx)) defIdx = DefaultOptionIndex();
                Orders[i + start].SelectedIndex = defIdx;
            }
        }

        // ===== PatternOptions 防呆：若空就從 JSON 補上 =====
        private void EnsurePatternOptionsFromJson()
        {
            if (PatternOptions.Count > 0 || _cfg == null) return;

            var optArr = _cfg["options"] as JArray;
            if (optArr == null) return;

            PatternOptions.Clear();
            foreach (var t in optArr)
            {
                var o = t as JObject; if (o == null) continue;
                int idx = o["index"] != null ? o["index"].Value<int>() : 0;
                string name = (string)o["name"] ?? idx.ToString();
                PatternOptions.Add(new PatternOption { Index = idx, Display = name });
            }
        }

        // ===== Helper =====
        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private bool OptionExists(int idx)
        {
            for (int i = 0; i < PatternOptions.Count; i++)
                if (PatternOptions[i].Index == idx) return true;
            return false;
        }

        private int DefaultOptionIndex()
        {
            return PatternOptions.Count > 0 ? PatternOptions[0].Index : 0;
        }

        private string LookupName(int idx)
        {
            for (int i = 0; i < PatternOptions.Count; i++)
                if (PatternOptions[i].Index == idx)
                    return PatternOptions[i].Display ?? idx.ToString();
            return idx.ToString();
        }

        // ===== 內部型別 =====
        public sealed class PatternOption
        {
            public int Index { get; set; }   // 0..63
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }
        }
    }
}

