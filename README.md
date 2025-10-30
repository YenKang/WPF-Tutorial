using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數 =====
        private const int TOTAL_MIN = 1;
        private const int TOTAL_MAX = 22;
        private const byte TOTAL_MASK = 0x1F; // [4:0]

        // ===== 狀態與旗標 =====
        private bool _preferHw = false;   // true → 切圖時讀暫存器覆蓋
        private JObject _cfg;

        // ===== Reg Map 名稱 =====
        private const string REG_TOTAL = "BIST_PT_TOTAL";
        private static string OrdName(int i) { return "BIST_PT_ORD" + i; }

        // ===== UI 綁定集合 =====
        public ObservableCollection<int> TotalOptions { get; private set; }
        public ObservableCollection<OrderSlot> Orders { get; private set; }
        public ObservableCollection<PatternOption> PatternOptions { get; private set; }

        private int _total;
        public int Total
        {
            get { return _total; }
            set
            {
                var v = Clamp(value, TOTAL_MIN, TOTAL_MAX);
                if (!SetValue(ref _total, v)) return;
                ResizeOrders(v);
            }
        }

        // === Command ===
        public ICommand ApplyCommand { get; private set; }

        public AutoRunVM()
        {
            Orders = new ObservableCollection<OrderSlot>();
            PatternOptions = new ObservableCollection<PatternOption>();
            TotalOptions = new ObservableCollection<int>();
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // =====================================================================
        // 1️⃣ 由 JSON 載入設定
        // =====================================================================
        public void LoadFrom(JObject node)
        {
            _cfg = node;

            // 1️⃣ 建立 Pattern 名稱表
            PatternOptions.Clear();
            if (node != null && node["options"] is JArray opts)
            {
                foreach (JObject o in opts)
                {
                    int idx = (int?)o["index"] ?? 0;
                    string name = (string)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }
            else
            {
                // fallback 預設 0..21
                for (int i = 0; i < 22; i++)
                {
                    PatternOptions.Add(new PatternOption { Index = i, Display = "Pattern " + i });
                }
            }

            // 2️⃣ 讀取 total 區塊
            int def = 22;
            int min = TOTAL_MIN, max = TOTAL_MAX;
            var totalObj = node != null ? node["total"] as JObject : null;
            if (totalObj != null)
            {
                def = (int?)totalObj["default"] ?? def;
                min = (int?)totalObj["min"] ?? min;
                max = (int?)totalObj["max"] ?? max;
            }

            // 重建 TotalOptions
            TotalOptions.Clear();
            for (int i = min; i <= max; i++) TotalOptions.Add(i);

            Total = Clamp(def, min, max);

            // 3️⃣ 初始化 Orders
            Orders.Clear();
            for (int i = 0; i < Total; i++)
                Orders.Add(new OrderSlot { DisplayNo = i + 1, SelectedIndex = i });

            // 4️⃣ 硬體優先：若 _preferHw = true, 從暫存器覆蓋
            if (_preferHw)
                TryRefreshFromHardware();
        }

        // =====================================================================
        // 2️⃣ 寫入暫存器（Set AutoRun）
        // =====================================================================
        private void Apply()
        {
            try
            {
                // 寫 TOTAL
                byte t = (byte)(Total & TOTAL_MASK);
                RegMap.Write8(REG_TOTAL, t);

                // 寫 ORD
                for (int i = 0; i < Orders.Count; i++)
                {
                    int idx = Clamp(Orders[i].SelectedIndex, 0, 21);
                    RegMap.Write8(OrdName(i), (byte)idx);
                }

                // 啟用硬體優先
                _preferHw = true;
                TryRefreshFromHardware();

                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply OK: Total=" + Total);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // =====================================================================
        // 3️⃣ 回讀暫存器覆寫 UI
        // =====================================================================
        private void RefreshFromHardware()
        {
            // 1) Total
            int rawTotal = RegMap.Read8(REG_TOTAL) & TOTAL_MASK;
            Total = Clamp(rawTotal, TOTAL_MIN, TOTAL_MAX);

            // 2) 每一張圖的 index
            for (int i = 0; i < Orders.Count; i++)
            {
                int idx = RegMap.Read8(OrdName(i)) & 0x3F;
                idx = Clamp(idx, 0, 21);
                Orders[i].SelectedIndex = idx;
            }
        }

        private void TryRefreshFromHardware()
        {
            try { RefreshFromHardware(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] HW refresh failed: " + ex.Message);
            }
        }

        // =====================================================================
        // 4️⃣ 小工具
        // =====================================================================
        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, TOTAL_MIN, TOTAL_MAX);

            // 增列
            while (Orders.Count < desired)
            {
                int n = Orders.Count;
                Orders.Add(new OrderSlot { DisplayNo = n + 1, SelectedIndex = n });
            }

            // 減列
            while (Orders.Count > desired)
            {
                Orders.RemoveAt(Orders.Count - 1);
            }

            // 重編號
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // =====================================================================
        // 5️⃣ 資料結構（C#7.3 風格）
        // =====================================================================
        public sealed class OrderSlot : ViewModelBase
        {
            private int _displayNo;
            private int _selectedIndex;

            public int DisplayNo
            {
                get { return _displayNo; }
                set { SetValue(ref _displayNo, value); }
            }

            public int SelectedIndex
            {
                get { return _selectedIndex; }
                set { SetValue(ref _selectedIndex, value); }
            }
        }

        public sealed class PatternOption : ViewModelBase
        {
            private int _index;
            private string _display;

            public int Index
            {
                get { return _index; }
                set { SetValue(ref _index, value); }
            }

            public string Display
            {
                get { return _display; }
                set { SetValue(ref _display, value); }
            }
        }
    }
}