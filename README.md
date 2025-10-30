using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private const int MaxOrders = 22;    // ORD0..ORD21
        private const int TotalMask = 0x1F;  // BIST_PT_TOTAL[4:0] = (Total-1)
        private const int OrdMask   = 0x3F;  // ORD 低6bit = pattern index

        private JObject _cfg;
        private bool _preferHw;
        private bool _isHydrating;

        // JSON 宣告的 min/max（經過夾限 1..22 後）；用來建表與擴充
        private int _totalMin = 1;
        private int _totalMax = MaxOrders;

        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                // 這裡不夾限，夾限在呼叫端（LoadFrom/Refresh）與 Ensure* 方法中做
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ========== 核心修正點：TotalOptions 與 Total 的安全流程 ==========
        private void BuildTotalOptions(int min, int max)
        {
            // 1..22 夾限，並保存到欄位（供後續擴充）
            _totalMin = Clamp(min, 1, MaxOrders);
            _totalMax = Clamp(max, 1, MaxOrders);
            if (_totalMin > _totalMax) { var t = _totalMin; _totalMin = _totalMax; _totalMax = t; }

            TotalOptions.Clear();
            for (int v = _totalMin; v <= _totalMax; v++)
                TotalOptions.Add(v);
        }

        // 確保清單「吃得下」指定的 total；硬體回讀可能 > JSON max，就擴到硬體值（不超過 22）
        private void EnsureTotalOptionsCovers(int required)
        {
            int needMax = Clamp(required, 1, MaxOrders);

            if (TotalOptions.Count == 0)
            {
                BuildTotalOptions(1, needMax);
                return;
            }

            int curMin = TotalOptions[0];
            int curMax = TotalOptions[TotalOptions.Count - 1];

            // 擴右邊
            if (needMax > curMax)
            {
                for (int v = curMax + 1; v <= needMax; v++)
                    TotalOptions.Add(v);
                _totalMax = needMax;
            }

            // 理論上不會需要擴左邊；保險維持從 1 起算
            if (curMin > 1)
            {
                TotalOptions.Clear();
                for (int v = 1; v <= _totalMax; v++)
                    TotalOptions.Add(v);
                _totalMin = 1;
            }
        }

        // ========== Public APIs ==========
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = _preferHw || preferHw;   // 一旦 true 就保留

            // 1) Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) PatternOptions 先建好（避免 Combo 失去 ItemsSource）
            EnsurePatternOptionsFromJson();

            // 3) TotalOptions：先建好清單，再決定 Total 來源
            var totalObj = autoRunControl["total"] as JObject;
            int jsonMin = (totalObj?["min"] != null) ? totalObj["min"].Value<int>() : 1;
            int jsonMax = (totalObj?["max"] != null) ? totalObj["max"].Value<int>() : MaxOrders;
            int jsonDef = (totalObj?["default"] != null) ? totalObj["default"].Value<int>() : MaxOrders;

            BuildTotalOptions(jsonMin, jsonMax); // 先建表（重點）

            _isHydrating = true;
            try
            {
                if (_preferHw)
                {
                    // 偏好硬體：Refresh 會先擴清單再設 Total，避免紅框
                    RefreshFromHardware();
                }
                else
                {
                    // JSON 模式：先把 default 夾回清單範圍，再設 Total
                    int def = Clamp(jsonDef, _totalMin, _totalMax);
                    Total = def;
                    HydrateOrdersFromJsonDefaultsForRange(0, def);
                }
            }
            finally
            {
                _isHydrating = false;
            }

            System.Diagnostics.Debug.WriteLine($"[AutoRun] LoadFrom done. preferHw={_preferHw}, TotalOptions={TotalOptions.Count}, Total={Total}");
        }

        public void Apply()
        {
            if (Total <= 0) return;

            int count = Clamp(Total, 1, MaxOrders);

            // TOTAL 寫入：(Total-1)
            byte totalRaw = (byte)((count - 1) & TotalMask);
            RegMap.Write8("BIST_PT_TOTAL", totalRaw);
            System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL raw=0x{totalRaw:X2} (UI={count})");

            // ORD 寫入
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

        public void RefreshFromHardware()
        {
            try
            {
                EnsurePatternOptionsFromJson();

                // 1) 讀 TOTAL，轉 UI 值（1..22）
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int totalHw = ((int)totalRaw & TotalMask) + 1;
                totalHw = Clamp(totalHw, 1, MaxOrders);

                // 2) 先確保清單吃得下硬體 total（重點：避免 SelectedValue 不在 ItemsSource）
                EnsureTotalOptionsCovers(totalHw);

                // 3) 再設 Total 並調整 slot
                _isHydrating = true;
                try
                {
                    Total = totalHw;
                    ResizeOrders(totalHw);
                }
                finally { _isHydrating = false; }

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL raw=0x{totalRaw:X2} → UI={totalHw}");

                // 4) 逐張讀 ORD
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
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ========== 內部邏輯 ==========
        private void ResizeOrders(int desired)
        {
            int need = Clamp(desired, 1, MaxOrders);
            int old = Orders.Count;

            while (Orders.Count < need)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            while (Orders.Count > need)
                Orders.RemoveAt(Orders.Count - 1);

            // 若偏好硬體，Total 從小變大時，僅回填新段（不動舊段）
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

            // 重設顯示序號（1-based）
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private void HydrateOrdersFromJsonDefaultsForRange(int start, int count)
        {
            if (_cfg == null) return;

            var ordersObj = _cfg["orders"] as JObject;
            var defArray = ordersObj != null ? ordersObj["defaults"] as JArray : null;
            if (defArray == null || defArray.Count == 0) // 沒提供 defaults 就用 options 順序
            {
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
            public int Index { get; set; }
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

