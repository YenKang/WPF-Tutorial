using System;
using System.Collections.ObjectModel;
using System.Linq;
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
        private const byte TOTAL_MASK = 0x1F; // [4:0] (HW: 0..21)
        private const byte ORD_MASK   = 0x3F; // [5:0] (HW: 0..63)

        // ===== 暫存器名稱（依你的 JSON registers 命名）=====
        private const string REG_TOTAL = "BIST_PT_TOTAL";
        private static string REG_ORD(int i) { return "BIST_PT_ORD" + i; } // 0..21

        // ===== 狀態 =====
        private bool _preferHw = false; // Set 後為 true：切回頁面時以硬體真值覆寫
        private JObject _cfg;

        // ===== 顯示用對照（index→name，僅作 log）=====
        private readonly System.Collections.Generic.Dictionary<int, string> _indexToName =
            new System.Collections.Generic.Dictionary<int, string>();

        // ====== 綁定屬性 ======
        private string _title;
        public string Title
        {
            get { return _title; }
            set { SetValue(ref _title, value); }
        }

        // Pattern Total 下拉清單（1..22 或 JSON 覆蓋）
        public ObservableCollection<int> TotalOptions { get; private set; }

        private int _total;
        // UI 顯示的總張數（1..22）
        public int Total
        {
            get { return _total; }
            set
            {
                int v = Clamp(value, TOTAL_MIN, TOTAL_MAX);
                if (!SetValue(ref _total, v)) return;
                ResizeOrders(v);
                // 若已經硬體優先，調整張數時同步回讀前 N 筆
                if (_preferHw) TryRefreshFromHardware();
            }
        }

        // 每一張圖的選擇（SelectedValue 綁這個 int，對應 PatternOptions[Index]）
        public ObservableCollection<OrderSlot> Orders { get; private set; }

        // Pattern 選項清單（來源 = JSON options）
        public ObservableCollection<PatternOption> PatternOptions { get; private set; }

        public ICommand ApplyCommand { get; private set; }

        public AutoRunVM()
        {
            Title = "Auto Run";
            TotalOptions   = new ObservableCollection<int>();
            Orders         = new ObservableCollection<OrderSlot>();
            PatternOptions = new ObservableCollection<PatternOption>();
            ApplyCommand   = CommandFactory.CreateCommand(Apply);
        }

        // =====================================================================
        // 1) 一開始點選 auto-run → 用 JSON 當作 default（Total 預設 22）
        // =====================================================================
        public void LoadFrom(JObject autoRunControl)
        {
            _cfg = autoRunControl;
            _preferHw = false; // 第一次進來以 JSON 為準

            // Title
            Title = _cfg != null && _cfg["title"] != null ? _cfg["title"].ToString() : "Auto Run";

            // 讀 options（index ↔ name）→ 作為 UI 顯示 & log 名稱
            PatternOptions.Clear();
            _indexToName.Clear();
            if (_cfg != null && _cfg["options"] is JArray opts)
            {
                foreach (JObject o in opts)
                {
                    int idx = (int?)o["index"] ?? 0;
                    string name = (string)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                    if (!_indexToName.ContainsKey(idx)) _indexToName.Add(idx, name);
                }
            }
            else
            {
                // fallback：至少補 0..63
                for (int i = 0; i < 64; i++)
                {
                    PatternOptions.Add(new PatternOption { Index = i, Display = i.ToString() });
                    if (!_indexToName.ContainsKey(i)) _indexToName.Add(i, i.ToString());
                }
            }

            // TotalOptions 與預設值
            int def = 22, min = TOTAL_MIN, max = TOTAL_MAX;
            var totalObj = _cfg != null ? _cfg["total"] as JObject : null;
            if (totalObj != null)
            {
                def = (int?)totalObj["default"] ?? def;
                min = (int?)totalObj["min"] ?? min;
                max = (int?)totalObj["max"] ?? max;
            }
            BuildTotalOptions(min, max);
            Total = Clamp(def, min, max);

            // 初始化 Orders：先依 defaults，如果沒有就用前 N 個遞增
            Orders.Clear();
            for (int i = 0; i < Total; i++)
                Orders.Add(new OrderSlot { DisplayNo = i + 1, SelectedIndex = 0 });

            var ordersObj = _cfg != null ? _cfg["orders"] as JObject : null;
            var defaults  = ordersObj != null ? ordersObj["defaults"] as JArray : null;
            if (defaults != null && defaults.Count > 0)
            {
                for (int i = 0; i < Total && i < defaults.Count; i++)
                {
                    int idx = (int?)defaults[i] ?? 0;
                    Orders[i].SelectedIndex = idx;
                }
            }
            else
            {
                for (int i = 0; i < Total; i++)
                {
                    int idx = i < PatternOptions.Count ? PatternOptions[i].Index : 0;
                    Orders[i].SelectedIndex = idx;
                }
            }
        }

        // =====================================================================
        // 3) Set：寫入 Total + ORD0..ORD(Total-1)（逐步印 Log），清掉尾巴
        // =====================================================================
        private void Apply()
        {
            try
            {
                // 1) 寫入 TOTAL（UI 1..22 → HW 0..21）
                int uiTotal = Clamp(Total, TOTAL_MIN, TOTAL_MAX);
                byte hwTotal = (byte)((uiTotal - 1) & TOTAL_MASK);
                RegMap.Write8(REG_TOTAL, hwTotal);
                System.Diagnostics.Debug.WriteLine(
                    "[AutoRun] Write TOTAL: UI=" + uiTotal + "  -> REG_TOTAL=0x" + hwTotal.ToString("X2"));

                // 2) 寫入有效的 ORD 槽位
                for (int i = 0; i < uiTotal; i++)
                {
                    int patternIndex = Orders[i].SelectedIndex;        // 0..63
                    byte val = (byte)(patternIndex & ORD_MASK);
                    RegMap.Write8(REG_ORD(i), val);

                    string name;
                    if (!_indexToName.TryGetValue(patternIndex, out name)) name = patternIndex.ToString();
                    System.Diagnostics.Debug.WriteLine(
                        "[AutoRun] Write ORD" + i.ToString("D2") + ": idx=" + patternIndex + " (" + name + ") -> 0x" + val.ToString("X2"));
                }

                // 3) 清掉後面沒用到的 ORD（避免殘值）
                for (int i = uiTotal; i < TOTAL_MAX; i++)
                {
                    RegMap.Write8(REG_ORD(i), 0x00); // 清成 0=Black
                }

                // 4) 之後切回頁面 → 以硬體為準
                _preferHw = true;

                // 5) 立即回讀確認同步（也印 Log）
                TryRefreshFromHardware();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // =====================================================================
        // 4/5) 切回頁面 / 調整張數後：回讀硬體（以真值覆寫 UI）
        // =====================================================================
        public void RefreshFromHardware()
        {
            if (!_preferHw) return;

            try
            {
                // 讀 TOTAL（HW 0..21 -> UI 1..22）
                int hwRaw = RegMap.Read8(REG_TOTAL) & TOTAL_MASK;
                int uiTotal = Clamp(hwRaw + 1, TOTAL_MIN, TOTAL_MAX);
                Total = uiTotal; // setter 會 ResizeOrders

                System.Diagnostics.Debug.WriteLine(
                    "[AutoRun] Read  TOTAL: UI=" + uiTotal + " (raw=0x" + hwRaw.ToString("X2") + ")");

                // 讀 ORD0..ORD(Total-1)
                for (int i = 0; i < uiTotal; i++)
                {
                    int v = RegMap.Read8(REG_ORD(i)) & ORD_MASK;
                    Orders[i].SelectedIndex = v;

                    string name;
                    if (!_indexToName.TryGetValue(v, out name)) name = v.ToString();
                    System.Diagnostics.Debug.WriteLine(
                        "[AutoRun] Read  ORD" + i.ToString("D2") + ": 0x" + v.ToString("X2") + " -> idx=" + v + " (" + name + ")");
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware failed: " + ex.Message);
            }
        }

        private void TryRefreshFromHardware()
        {
            try { RefreshFromHardware(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] HW refresh skipped: " + ex.Message);
            }
        }

        // =====================================================================
        // 小工具
        // =====================================================================
        private void BuildTotalOptions(int min, int max)
        {
            int lo = Math.Max(TOTAL_MIN, min);
            int hi = Math.Min(TOTAL_MAX, max);
            TotalOptions.Clear();
            for (int i = lo; i <= hi; i++) TotalOptions.Add(i);
        }

        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, TOTAL_MIN, TOTAL_MAX);

            // 增列
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            // 減列
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            // 重新編號（1-based）
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
        // 內部型別（C# 7.3 風格）
        // =====================================================================
        public sealed class OrderSlot : ViewModelBase
        {
            private int _displayNo;
            private int _selectedIndex; // 真正要寫進 ORD 的 index（0..63）

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
            private int _index;     // 0..63
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