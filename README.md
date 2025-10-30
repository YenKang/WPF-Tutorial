using System;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private bool _preferHw = false; // Set 過後改讀硬體
        private JObject _cfg;

        // === 暫存器對應 ===
        private string _totalTarget = "";             // BIST_PT_TOTAL
        private string[] _orderTargets = new string[0]; // BIST_PT_ORD0~ORD21

        // === Pattern Total Combo ===
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();

        private int _total;
        public int Total
        {
            get => _total;
            set
            {
                if (SetValue(ref _total, value))
                    ResizeOrders(_total);
            }
        }

        // === Orders ===
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // === Pattern 選項清單 ===
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();

        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ========== 由 JSON 載入設定 ==========
        public void LoadFrom(JObject node)
        {
            _cfg = node;
            if (_cfg == null) return;

            // 1️⃣ 讀 Pattern Title (optional)
            Title = (string?)_cfg["title"] ?? "Auto Run";

            // 2️⃣ 讀 total
            var totalObj = _cfg["total"] as JObject;
            _totalTarget = (string?)totalObj?["target"] ?? "BIST_PT_TOTAL";
            int min = (int?)totalObj?["min"] ?? 1;
            int max = (int?)totalObj?["max"] ?? 22;
            int def = (int?)totalObj?["default"] ?? 22;

            TotalOptions.Clear();
            for (int i = min; i <= max; i++) TotalOptions.Add(i);
            Total = Clamp(def, min, max);

            // 3️⃣ 讀 orders
            var ordersObj = _cfg["orders"] as JObject;
            _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? new string[0];
            Orders.Clear();
            for (int i = 0; i < Total; i++) Orders.Add(new OrderSlot { DisplayNo = i + 1 });

            // 4️⃣ 讀 options (index-name 對照)
            PatternOptions.Clear();
            if (_cfg["options"] is JArray arr)
            {
                foreach (JObject o in arr.OfType<JObject>())
                {
                    int idx = (int?)o["index"] ?? 0;
                    string name = (string?)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // 5️⃣ 若曾 Set 過 → 改用硬體回讀覆寫
            if (_preferHw)
            {
                try { RefreshFromRegister(); }
                catch (Exception ex) { System.Diagnostics.Debug.WriteLine("[AutoRun] skip refresh: " + ex.Message); }
            }
        }

        // ========== 寫入暫存器 (Set AutoRun) ==========
        private void Apply()
        {
            if (_cfg == null) return;

            try
            {
                // 寫入總數
                if (!string.IsNullOrWhiteSpace(_totalTarget))
                    RegMap.Write8(_totalTarget, (byte)(Total & 0x1F));

                // 寫入各個 ORD pattern
                for (int i = 0; i < Orders.Count && i < _orderTargets.Length; i++)
                {
                    var tgt = _orderTargets[i];
                    if (string.IsNullOrWhiteSpace(tgt)) continue;

                    int idx = Orders[i].SelectedIndex & 0x3F; // 6 bits
                    RegMap.Write8(tgt, (byte)idx);
                }

                // ✅ 寫入完成 → 啟用硬體模式
                _preferHw = true;
                RefreshFromRegister();

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Apply OK: Total={Total}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // ========== 從暫存器回讀 ==========
        public void RefreshFromRegister()
        {
            if (!_preferHw) return;

            // 1️⃣ 讀取總數
            if (!string.IsNullOrWhiteSpace(_totalTarget))
            {
                int raw = RegMap.Read8(_totalTarget) & 0x1F;
                if (TotalOptions.Count > 0)
                {
                    int min = TotalOptions[0];
                    int max = TotalOptions[TotalOptions.Count - 1];
                    Total = Math.Max(min, Math.Min(max, raw));
                }
                else
                {
                    Total = raw;
                }
            }

            // 2️⃣ 逐一回讀 ORDx 值
            for (int i = 0; i < Orders.Count && i < _orderTargets.Length; i++)
            {
                var tgt = _orderTargets[i];
                if (string.IsNullOrWhiteSpace(tgt)) continue;

                int idx = RegMap.Read8(tgt) & 0x3F;
                Orders[i].SelectedIndex = idx;

                // debug log
                var match = PatternOptions.FirstOrDefault(p => p.Index == idx);
                string name = match?.Display ?? idx.ToString();
                System.Diagnostics.Debug.WriteLine($"[AutoRun] ORD{i:D2}={idx:D2} ({name})");
            }
        }

        // ========== 小工具 ==========
        private static int Clamp(int v, int min, int max)
            => v < min ? min : (v > max ? max : v);

        private void ResizeOrders(int desired)
        {
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1 });
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);
        }

        // === 子類別 ===
        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo { get => GetValue<int>(); set => SetValue(value); }
            public int SelectedIndex { get => GetValue<int>(); set => SetValue(value); }
        }

        public sealed class PatternOption
        {
            public int Index { get; set; }
            public string Display { get; set; }
        }

        public string Title { get => GetValue<string>(); set => SetValue(value); }
    }
}