using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數 / 掩碼 =====
        private const int MaxOrders  = 22;   // 有效 ORD: BIST_PT_ORD0 .. BIST_PT_ORD21
        private const byte TotalMask = 0x1F; // TOTAL 寄存器低 5 bits：硬體存 (Total-1)
        private const byte OrdMask   = 0x3F; // ORD 低 6 bits：pattern index 0..63

        // ===== 狀態 =====
        private JObject _cfg;
        private bool _preferHw;     // true：偏好硬體真值；false：偏好 JSON default
        private bool _isHydrating;  // 內部回填旗標，避免 setter 互觸

        // ===== 綁定集合（固定實例，切勿 new）=====
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ===== 綁定屬性 =====
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
                int old = GetValue<int>();
                int v = Clamp(value, 1, MaxOrders);
                if (!SetValue(v)) return;

                // 只調整數量
                ResizeOrders(v);

                // 依來源：只補「新增的區段」
                if (_preferHw && v > old)
                {
                    // 硬體模式：讀 ORD[old..v-1] 回填
                    FillOrdersFromHardwareRange(old, v);
                }
                else if (!_preferHw && v > old)
                {
                    // JSON 模式：用 JSON default 回填新增區段
                    HydrateOrdersFromJsonDefaultsForRange(old, v);
                }
            }
        }

        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // preferHw：外部第一次載入/切回頁面時可要求偏好硬體；_preferHw = _preferHw || preferHw（不會被覆蓋回 false）
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = _preferHw || preferHw; // 一旦 true 就保留

            // 1) Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) PatternOptions（僅 Clear/Add，不 new）
            BuildPatternOptionsFromJson(autoRunControl);

            // 3) TotalOptions（以 JSON min/max 建立，但卡在 1..MaxOrders）
            var totalObj = autoRunControl["total"] as JObject;
            int min = Clamp(totalObj?["min"]?.Value<int>() ?? 1, 1, MaxOrders);
            int max = Clamp(totalObj?["max"]?.Value<int>() ?? MaxOrders, 1, MaxOrders);
            if (min > max) { var t = min; min = max; max = t; }

            TotalOptions.Clear();
            for (int i = min; i <= max; i++) TotalOptions.Add(i);

            // 4) 來源決策
            if (_preferHw)
            {
                // 偏好硬體：直接回讀，完全跳過 JSON default
                RefreshFromHardware();
            }
            else
            {
                // JSON 模式：以 default 初始化
                int def = Clamp(totalObj?["default"]?.Value<int>() ?? MaxOrders, min, max);

                _isHydrating = true;
                try
                {
                    // 設定 Total（只調數量）
                    SetValue(nameof(Total), def);
                    ResizeOrders(def);
                    // 以 JSON default 回填全部
                    HydrateOrdersFromJsonDefaultsForRange(0, def);
                }
                finally { _isHydrating = false; }
            }
        }

        // ===== Apply：寫入 registers，切硬體模式，並回讀同步 =====
        public void Apply()
        {
            if (Total <= 0) return;

            int count = Clamp(Total, 1, MaxOrders);

            // 1) 寫 TOTAL（UI 1..22 → raw 0..21）
            byte totalRaw = (byte)((count - 1) & TotalMask);
            RegMap.Write8("BIST_PT_TOTAL", totalRaw);
            System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL raw=0x{totalRaw:X2} (UI={count})");

            // 2) 寫 ORD0..ORD{count-1}
            for (int i = 0; i < count && i < Orders.Count; i++)
            {
                string reg = GetOrdRegName(i);
                if (reg == null) break;

                byte idxRaw = (byte)(Orders[i].SelectedIndex & OrdMask);
                RegMap.Write8(reg, idxRaw);
                System.Diagnostics.Debug.WriteLine($"[AutoRun] Write ORD{i:00} = idx={idxRaw} ({LookupName(idxRaw)})");
            }

            _preferHw = true;   // 往後以硬體真值為主
            RefreshFromHardware();
        }

        // ===== 從硬體完整回讀（不改硬體 TOTAL；只用它來呈現 UI）=====
        public void RefreshFromHardware()
        {
            try
            {
                // 1) 讀 TOTAL，夾回 1..MaxOrders
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int total = Clamp(((int)totalRaw & TotalMask) + 1, 1, MaxOrders);

                _isHydrating = true;
                try
                {
                    // 先把 UI slot 調成硬體張數（避免越界）
                    SetValue(nameof(Total), total);
                    ResizeOrders(total);
                }
                finally { _isHydrating = false; }

                // 2) 逐一讀 ORD0..ORD{total-1}
                for (int i = 0; i < total; i++)
                {
                    string reg = GetOrdRegName(i);
                    if (reg == null) break;

                    byte idxRaw = RegMap.Read8(reg);
                    int idx = idxRaw & OrdMask;

                    Orders[i].SelectedIndex = OptionExists(idx) ? idx : DefaultOptionIndex();
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Read ORD{i:00}: 0x{idxRaw:X2} -> idx={idx} ({LookupName(idx)})");
                }

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL: raw=0x{totalRaw:X2} -> UI={total}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ===== 調整 slot 數量（不決定來源）=====
        private void ResizeOrders(int desired)
        {
            int need = Clamp(desired, 1, MaxOrders);

            while (Orders.Count < need)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1 });

            while (Orders.Count > need)
                Orders.RemoveAt(Orders.Count - 1);

            // 重編號（1-based）
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        // ===== 當 Total 變大時，只讀新增區段的硬體 ORD =====
        private void FillOrdersFromHardwareRange(int start, int endExclusive)
        {
            for (int i = start; i < endExclusive && i < Orders.Count && i < MaxOrders; i++)
            {
                string reg = GetOrdRegName(i);
                if (reg == null) break;

                byte idxRaw = RegMap.Read8(reg);
                int idx = idxRaw & OrdMask;
                Orders[i].SelectedIndex = OptionExists(idx) ? idx : DefaultOptionIndex();

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Fill ORD{i:00} (partial) = 0x{idxRaw:X2} -> idx={idx} ({LookupName(idx)})");
            }
        }

        // ===== JSON 模式：只為新增區段回填 defaults =====
        private void HydrateOrdersFromJsonDefaultsForRange(int start, int endExclusive)
        {
            int count = Clamp(endExclusive, 1, MaxOrders);
            var defaults = GetJsonDefaultOrderIndices(count);

            for (int i = start; i < endExclusive && i < Orders.Count; i++)
            {
                int idx = (i < defaults.Length) ? defaults[i] : DefaultOptionIndex();
                Orders[i].SelectedIndex = idx;
            }
        }

        // ===== 由 JSON 構建 PatternOptions =====
        private void BuildPatternOptionsFromJson(JObject cfg)
        {
            PatternOptions.Clear();
            var arr = cfg?["options"] as JArray;
            if (arr == null) return;

            foreach (var t in arr)
            {
                var o = t as JObject; if (o == null) continue;
                int idx = o["index"]?.Value<int>() ?? 0;
                string name = (string)o["name"] ?? idx.ToString();
                PatternOptions.Add(new PatternOption { Index = idx, Display = name });
            }
        }

        // ===== 取得 JSON 預設順序（orders.defaults）；若沒有就用 options 順序 =====
        private int[] GetJsonDefaultOrderIndices(int count)
        {
            count = Clamp(count, 1, MaxOrders);

            var ordersObj = _cfg?["orders"] as JObject;
            var defaults = ordersObj?["defaults"] as JArray;

            if (defaults != null && defaults.Count > 0)
            {
                int n = Math.Min(count, defaults.Count);
                var arr = new int[n];
                for (int i = 0; i < n; i++)
                    arr[i] = defaults[i].Value<int>();
                return arr;
            }

            // fallback：依 options 順序
            var seq = new int[count];
            for (int i = 0; i < count; i++)
                seq[i] = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
            return seq;
        }

        // ===== 工具 =====
        private static string GetOrdRegName(int i)
        {
            if (i < 0 || i >= MaxOrders) return null;
            return $"BIST_PT_ORD{i}";
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

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // ===== 內部型別 =====
        public sealed class PatternOption
        {
            public int Index { get; set; }   // 0..63
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo { get { return GetValue<int>(); } set { SetValue(value); } }
            public int SelectedIndex { get { return GetValue<int>(); } set { SetValue(value); } }
        }
    }

    // 你的 RegMap 應已有實作；若沒有，可依專案版本替換/刪除。
    internal static class RegMap
    {
        public static byte Read8(string name) { /* your impl */ return 0; }
        public static void Write8(string name, byte value) { /* your impl */ }
        public static void Rmw8(string name, byte mask, byte value)
        {
            // 典型 RMW 範例（若需要）
            byte cur = Read8(name);
            cur = (byte)((cur & ~mask) | (value & mask));
            Write8(name, cur);
        }
    }
}