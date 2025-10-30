using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private const int MaxOrders = 22;
        private const int TotalMask = 0x1F;
        private const int OrdMask   = 0x3F;

        private JObject _cfg;
        private bool _preferHw;
        private bool _isHydrating;

        // === UI 綁定 ===
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
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        public ICommand ApplyCommand { get; }

        // === 建構子 ===
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // === 載入 JSON ===
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null)
                return;

            _cfg = autoRunControl;
            _preferHw = _preferHw || preferHw;

            // 1️⃣ Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2️⃣ Pattern options
            EnsurePatternOptionsFromJson();

            // 3️⃣ Total ComboBox
            var totalObj = autoRunControl["total"] as JObject;
            int min = Clamp(totalObj?["min"]?.Value<int>() ?? 1, 1, MaxOrders);
            int max = Clamp(totalObj?["max"]?.Value<int>() ?? MaxOrders, 1, MaxOrders);
            int def = Clamp(totalObj?["default"]?.Value<int>() ?? MaxOrders, min, max);

            TotalOptions.Clear();
            for (int i = min; i <= max; i++)
                TotalOptions.Add(i);

            _isHydrating = true;
            try
            {
                if (_preferHw)
                {
                    RefreshFromHardware();
                }
                else
                {
                    Total = def;
                    HydrateOrdersFromJsonDefaultsForRange(0, def);
                }
            }
            finally
            {
                _isHydrating = false;
            }

            System.Diagnostics.Debug.WriteLine($"[AutoRun] LoadFrom done. preferHw={_preferHw}");
        }

        // === 寫入 ===
        public void Apply()
        {
            if (Total <= 0) return;

            // 寫 TOTAL: 0=1張
            byte totalRaw = (byte)((Total - 1) & TotalMask);
            RegMap.Rmw8("BIST_PT_TOTAL", TotalMask, totalRaw);
            System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL = {totalRaw:X2}");

            // 寫 ORD0..ORDn
            for (int i = 0; i < Total && i < Orders.Count; i++)
            {
                string reg = "BIST_PT_ORD" + i;
                byte val = (byte)(Orders[i].SelectedIndex & OrdMask);
                RegMap.Write8(reg, val);
                System.Diagnostics.Debug.WriteLine($"[AutoRun] Write {reg} = 0x{val:X2} ({LookupName(val)})");
            }

            _preferHw = true;
            RefreshFromHardware();
        }

        // === 回讀暫存器 ===
        public void RefreshFromHardware()
        {
            try
            {
                EnsurePatternOptionsFromJson();

                // 1️⃣ 讀取 TOTAL
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int total = ((int)totalRaw & TotalMask) + 1;
                total = Clamp(total, 1, MaxOrders);

                _isHydrating = true;
                try
                {
                    Total = total;
                    ResizeOrders(total);
                }
                finally { _isHydrating = false; }

                // 2️⃣ 讀取每張圖順序
                for (int i = 0; i < total; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte val = RegMap.Read8(reg);
                    int idx = val & OrdMask;

                    if (!OptionExists(idx))
                        idx = 0;

                    Orders[i].SelectedIndex = idx;

                    System.Diagnostics.Debug.WriteLine($"[AutoRun] ORD{i:D2} = 0x{val:X2} ({LookupName(idx)})");
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware done.");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // === Resize ===
        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, 1, MaxOrders);
            int old = Orders.Count;

            // grow
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            // shrink
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            // 若 preferHw：讀新 slot 的硬體值
            if (_preferHw && desired > old)
            {
                for (int i = old; i < desired; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    try
                    {
                        byte val = RegMap.Read8(reg);
                        int idx = val & OrdMask;
                        if (!OptionExists(idx)) idx = 0;
                        Orders[i].SelectedIndex = idx;

                        System.Diagnostics.Debug.WriteLine(
                            $"[AutoRun] Resize new slot ORD{i:D2} <- 0x{val:X2} ({LookupName(idx)})");
                    }
                    catch
                    {
                        Orders[i].SelectedIndex = 0;
                    }
                }
            }
        }

        // === JSON 預設填入 ===
        private void HydrateOrdersFromJsonDefaultsForRange(int start, int count)
        {
            if (_cfg == null) return;

            var ordersObj = _cfg["orders"] as JObject;
            if (ordersObj == null) return;

            var defArray = ordersObj["defaults"] as JArray;
            if (defArray == null || defArray.Count == 0) return;

            for (int i = 0; i < count && (i + start) < Orders.Count; i++)
            {
                int defIdx = defArray.Count > (i + start)
                    ? defArray[i + start].Value<int>()
                    : 0;

                if (defIdx < 0 || defIdx >= PatternOptions.Count)
                    defIdx = 0;

                Orders[i + start].SelectedIndex = defIdx;
            }
        }

        // === Helper ===
        private void EnsurePatternOptionsFromJson()
        {
            if (PatternOptions.Count > 0 || _cfg == null) return;

            var optArr = _cfg["options"] as JArray;
            if (optArr == null) return;

            PatternOptions.Clear();
            foreach (var t in optArr)
            {
                var o = t as JObject; if (o == null) continue;
                int idx = o["index"]?.Value<int>() ?? 0;
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
            foreach (var p in PatternOptions)
                if (p.Index == idx)
                    return true;
            return false;
        }

        private string LookupName(int idx)
        {
            foreach (var p in PatternOptions)
                if (p.Index == idx)
                    return p.Display ?? idx.ToString();
            return idx.ToString();
        }

        // === Inner Types ===
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