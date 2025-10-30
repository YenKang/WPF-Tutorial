using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數 =====
        private const int MaxOrders = 22;    // ORD0..ORD21
        private const int TotalMask = 0x1F;  // BIST_PT_TOTAL[4:0] = (Total-1)
        private const int OrdMask   = 0x3F;  // ORD 低6bit = pattern index

        // ===== 狀態 =====
        private JObject _cfg;
        private bool _preferHw;
        private bool _isHydrating;
        private int _totalMin = 1;           // JSON 宣告的 min/max（夾 1..22）
        private int _totalMax = MaxOrders;

        // ===== 綁定集合（永遠不 new）=====
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ===== 綁定屬性 =====
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        // Total 僅負責觸發 Resize；安全性由呼叫端確保（SafeSetTotal）
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (value < 1) value = 1;
                if (value > MaxOrders) value = MaxOrders;
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        public ICommand ApplyCommand { get; }

        // ===== 建構子：先撐起 ItemsSource，避免初載紅框 =====
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            if (TotalOptions.Count == 0)
            {
                for (int v = 1; v <= MaxOrders; v++) TotalOptions.Add(v);
            }
            if (Total == 0) Total = 1; // SelectedValue 一開始就合法
        }

        // ===== 載入 JSON（穩定順序 + 延後設定）=====
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = _preferHw || preferHw;  // 一旦 true 就保留
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 1) 先建 PatternOptions（避免內層 Combo 無 ItemsSource）
            EnsurePatternOptionsFromJson();

            // 2) 先建 TotalOptions，再決定要設多少
            var totalObj = autoRunControl["total"] as JObject;
            int jsonMin = totalObj != null && totalObj["min"] != null ? totalObj["min"].Value<int>() : 1;
            int jsonMax = totalObj != null && totalObj["max"] != null ? totalObj["max"].Value<int>() : MaxOrders;
            int jsonDef = totalObj != null && totalObj["default"] != null ? totalObj["default"].Value<int>() : MaxOrders;
            BuildTotalOptions(jsonMin, jsonMax); // ★ 永遠先把清單建好

            // 3) 用 Dispatcher 延後到 UI 綁定完成才設 Total（根治紅框）
            System.Windows.Application.Current.Dispatcher.BeginInvoke(
                new Action(() =>
                {
                    _isHydrating = true;
                    try
                    {
                        if (_preferHw)
                        {
                            RefreshFromHardware(); // 內部 SafeSetTotal → 不紅
                        }
                        else
                        {
                            int def = Clamp(jsonDef, _totalMin, _totalMax);
                            SafeSetTotal(def);                     // SelectedValue 安全
                            HydrateOrdersFromJsonDefaultsForRange(0, def);
                        }
                    }
                    finally { _isHydrating = false; }

                    System.Diagnostics.Debug.WriteLine(
                        $"[AutoRun] LoadFrom deferred set done. preferHw={_preferHw}, Total={Total}, TotalOptions={TotalOptions.Count}");
                }),
                System.Windows.Threading.DispatcherPriority.Loaded
            );
        }

        // ===== 寫入暫存器 =====
        public void Apply()
        {
            if (Total <= 0) return;

            int count = Clamp(Total, 1, MaxOrders);

            // 寫 TOTAL（0=1張）
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

        // ===== 回讀暫存器（先確保清單 → 再 SafeSetTotal）=====
        public void RefreshFromHardware()
        {
            try
            {
                EnsurePatternOptionsFromJson();

                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int totalHw = Clamp(((int)totalRaw & TotalMask) + 1, 1, MaxOrders);

                _isHydrating = true;
                try
                {
                    EnsureTotalOptionsCovers(totalHw); // 先保證 ItemsSource 吃得下
                    SafeSetTotal(totalHw);             // 再設 SelectedValue（不紅）
                    ResizeOrders(totalHw);
                }
                finally { _isHydrating = false; }

                // 逐張讀 ORD
                for (int i = 0; i < totalHw; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte val = RegMap.Read8(reg);
                    int idx = val & OrdMask;
                    if (!OptionExists(idx)) idx = DefaultOptionIndex();
                    Orders[i].SelectedIndex = idx;

                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Read {reg} = 0x{val:X2} → idx={idx} ({LookupName(idx)})");
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware done.");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ===== 只 Clear/Add，不 new =====
        private void BuildTotalOptions(int min, int max)
        {
            _totalMin = Clamp(min, 1, MaxOrders);
            _totalMax = Clamp(max, 1, MaxOrders);
            if (_totalMin > _totalMax) { var t = _totalMin; _totalMin = _totalMax; _totalMax = t; }

            TotalOptions.Clear();
            for (int v = _totalMin; v <= _totalMax; v++)
                TotalOptions.Add(v);
        }

        // 硬體 total 超過 JSON 上限時，把清單擴到硬體值（不超過 22）
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
                for (int v = curMax + 1; v <= needMax; v++) TotalOptions.Add(v);
                _totalMax = needMax;
            }

            // 保險：若不是 1 起算就重建（理論上不會）
            if (curMin > 1)
            {
                TotalOptions.Clear();
                for (int v = 1; v <= _totalMax; v++) TotalOptions.Add(v);
                _totalMin = 1;
            }
        }

        // 設定 Total 的唯一入口：先確保清單，後設值（可避免任何紅框）
        private void SafeSetTotal(int v)
        {
            v = Clamp(v, 1, MaxOrders);

            if (TotalOptions.Count == 0)
            {
                // 極端情境（切頁 race）：延後到本輪 UI 完成後再設
                System.Windows.Application.Current.Dispatcher.BeginInvoke(
                    new Action(() =>
                    {
                        EnsureTotalOptionsCovers(v);
                        Total = v;
                        System.Diagnostics.Debug.WriteLine($"[AutoRun] (deferred) SafeSetTotal={Total}, TotalOptions={TotalOptions.Count}");
                    }),
                    System.Windows.Threading.DispatcherPriority.Background
                );
                return;
            }

            EnsureTotalOptionsCovers(v);
            Total = v;
            System.Diagnostics.Debug.WriteLine($"[AutoRun] SafeSetTotal={Total}, TotalOptions={TotalOptions.Count}");
        }

        // Resize（_preferHw=true 時，新增加的 slot 從硬體補值）
        private void ResizeOrders(int desired)
        {
            int need = Clamp(desired, 1, MaxOrders);
            int old = Orders.Count;

            while (Orders.Count < need)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            while (Orders.Count > need)
                Orders.RemoveAt(Orders.Count - 1);

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

            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1; // 1-based
        }

        // JSON 預設填入（第一次載入）
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