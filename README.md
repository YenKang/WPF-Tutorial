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
        // ===== 常數 / 掩碼 =====
        private const int TOTAL_MIN  = 1;
        private const int TOTAL_MAX  = 22;
        private const byte TOTAL_MASK = 0x1F; // [4:0]  -> 0..21 (硬體儲存的是 Total-1)
        private const byte ORD_MASK   = 0x3F; // [5:0]  -> 0..63

        // ===== 暫存器名稱（依你的 JSON registers 命名）=====
        private const string REG_TOTAL = "BIST_PT_TOTAL";
        private static string REG_ORD(int i) { return "BIST_PT_ORD" + i; } // BIST_PT_ORD0..21

        // ===== 內部狀態 =====
        private JObject _cfg;
        private bool _preferHw = false; // Set 之後改為 true：之後切換頁面都讀硬體
        private readonly System.Collections.Generic.Dictionary<int, string> _indexToName =
            new System.Collections.Generic.Dictionary<int, string>(); // 只用於 Debug Log 顯示用名

        // ===== 綁定屬性 =====
        private string _title = "Auto Run";
        public string Title
        {
            get { return _title; }
            set { SetValue(ref _title, value); }
        }

        public ObservableCollection<int> TotalOptions { get; private set; }
        public ObservableCollection<OrderSlot> Orders { get; private set; }
        public ObservableCollection<PatternOption> PatternOptions { get; private set; }

        private int _total = 22; // UI 顯示的總張數（1..22）
        public int Total
        {
            get { return _total; }
            set
            {
                int v = Clamp(value, TOTAL_MIN, TOTAL_MAX);
                if (!SetValue(ref _total, v)) return;
                ResizeOrders(v);
                // 若已經切到硬體模式，改變張數時也會把前 N 筆從硬體回讀
                if (_preferHw) TryRefreshFromHardware();
            }
        }

        public ICommand ApplyCommand { get; private set; }

        public AutoRunVM()
        {
            TotalOptions   = new ObservableCollection<int>();
            Orders         = new ObservableCollection<OrderSlot>();
            PatternOptions = new ObservableCollection<PatternOption>();
            ApplyCommand   = CommandFactory.CreateCommand(Apply);

            // 給 UI 一個安全的初始狀態，避免剛載入就空集合
            BuildTotalOptions(TOTAL_MIN, TOTAL_MAX);
            ResizeOrders(_total);
            EnsurePatternOptionsNotEmpty();
        }

        // =====================================================================
        // 1) 進入 Auto-Run 頁：第一次吃 JSON；若已 Set 過（_preferHw）→直接讀硬體
        // =====================================================================
        public void LoadFrom(JObject autoRunControl)
        {
            _cfg = autoRunControl;

            // 1) Title（不論是否硬體模式都可更新）
            Title = (_cfg != null && _cfg["title"] != null) ? _cfg["title"].ToString() : "Auto Run";

            // 2) 讀 options（index↔name）→ 供 UI 顯示與 log
            PatternOptions.Clear();
            _indexToName.Clear();
            if (_cfg != null && _cfg["options"] is JArray opts)
            {
                foreach (var t in opts.OfType<JObject>())
                {
                    int idx   = (int?)t["index"] ?? 0;
                    string nm = (string)t["name"]  ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = nm });
                    if (!_indexToName.ContainsKey(idx)) _indexToName.Add(idx, nm);
                }
            }
            else
            {
                EnsurePatternOptionsNotEmpty();
            }

            // 3) 先依 JSON 準備 TotalOptions（但先不動 Total）
            int def = 22, min = TOTAL_MIN, max = TOTAL_MAX;
            var totalObj = (_cfg != null) ? _cfg["total"] as JObject : null;
            if (totalObj != null)
            {
                def = (int?)totalObj["default"] ?? def;
                min = (int?)totalObj["min"] ?? min;
                max = (int?)totalObj["max"] ?? max;
            }
            BuildTotalOptions(min, max);

            // 4) 若已經硬體模式：直接讀硬體覆寫（跳過 JSON 初始化）
            if (_preferHw)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] LoadFrom(): preferHw=true → refresh from HW");
                TryRefreshFromHardware();
                return;
            }

            // 5) 第一次進來或 Reset 後：依 JSON default 初始化
            System.Diagnostics.Debug.WriteLine("[AutoRun] LoadFrom(): preferHw=false → use JSON defaults");

            int uiDefault = Clamp(def, min, max);
            Total = uiDefault; // setter 會 ResizeOrders

            // 初始化 Orders（先清空再建 N 筆）
            Orders.Clear();
            for (int i = 0; i < Total; i++)
                Orders.Add(new OrderSlot { DisplayNo = i + 1, SelectedIndex = 0 });

            var ordersObj = (_cfg != null) ? _cfg["orders"] as JObject : null;
            var defaults  = (ordersObj != null) ? ordersObj["defaults"] as JArray : null;
            if (defaults != null && defaults.Count > 0)
            {
                for (int i = 0; i < Total && i < defaults.Count; i++)
                    Orders[i].SelectedIndex = (int?)defaults[i] ?? 0;
            }
            else
            {
                for (int i = 0; i < Total; i++)
                    Orders[i].SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
            }
        }

        // =====================================================================
        // 2) Set：寫入 Total 與 ORD0..ORD(Total-1)；清尾巴；切硬體模式；回讀確認
        // =====================================================================
        private void Apply()
        {
            try
            {
                int uiTotal = Clamp(Total, TOTAL_MIN, TOTAL_MAX);
                byte hwTotal = (byte)((uiTotal - 1) & TOTAL_MASK);

                // 寫 TOTAL（UI 1..22 → HW 0..21）
                RegMap.Write8(REG_TOTAL, hwTotal);
                System.Diagnostics.Debug.WriteLine("[AutoRun] Write TOTAL: UI=" + uiTotal + " -> 0x" + hwTotal.ToString("X2"));

                // 寫 ORD（前 N 筆）
                for (int i = 0; i < uiTotal; i++)
                {
                    int patIdx = 0;
                    if (i >= 0 && i < Orders.Count)
                        patIdx = Orders[i].SelectedIndex;

                    byte ordVal = (byte)(patIdx & ORD_MASK);
                    RegMap.Write8(REG_ORD(i), ordVal);

                    string name;
                    if (!_indexToName.TryGetValue(patIdx, out name)) name = patIdx.ToString();
                    System.Diagnostics.Debug.WriteLine("[AutoRun] Write ORD" + i.ToString("D2") + ": idx=" + patIdx + " (" + name + ") -> 0x" + ordVal.ToString("X2"));
                }

                // 清尾巴（避免殘值被硬體取到）
                for (int i = uiTotal; i < TOTAL_MAX; i++)
                    RegMap.Write8(REG_ORD(i), 0x00);

                // 切到硬體模式，之後切回頁面都讀硬體
                _preferHw = true;

                // 立即回讀同步（保險）
                TryRefreshFromHardware();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // =====================================================================
        // 3) 切回頁面 / 調整張數後：若硬體模式，讀回 Total + 各 ORD 覆寫 UI
        // =====================================================================
        public void RefreshFromHardware()
        {
            if (!_preferHw) return;

            try
            {
                // 硬體 0..21 → UI 1..22
                int hwRaw   = RegMap.Read8(REG_TOTAL) & TOTAL_MASK;
                int uiTotal = Clamp(hwRaw + 1, TOTAL_MIN, TOTAL_MAX);

                EnsureTotalOptionsCover(uiTotal);

                // 指派 Total（會 ResizeOrders，再次防越界）
                Total = uiTotal;

                // 回讀 ORD0..ORD(Total-1)
                for (int i = 0; i < Total; i++)
                {
                    int v = RegMap.Read8Safe(REG_ORD(i)) & ORD_MASK; // 若你沒有 Read8Safe，就改回 Read8
                    if (i >= 0 && i < Orders.Count)
                        Orders[i].SelectedIndex = v;

                    string name;
                    if (!_indexToName.TryGetValue(v, out name)) name = v.ToString();
                    System.Diagnostics.Debug.WriteLine("[AutoRun] Read  ORD" + i.ToString("D2") + ": 0x" + v.ToString("X2") + " -> idx=" + v + " (" + name + ")");
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] Read  TOTAL: UI=" + uiTotal + " (raw=0x" + hwRaw.ToString("X2") + ")");
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
        private void EnsurePatternOptionsNotEmpty()
        {
            if (PatternOptions.Count > 0) return;
            for (int i = 0; i < 64; i++)
            {
                PatternOptions.Add(new PatternOption { Index = i, Display = i.ToString() });
                if (!_indexToName.ContainsKey(i)) _indexToName.Add(i, i.ToString());
            }
        }

        private void BuildTotalOptions(int min, int max)
        {
            int lo = Math.Max(TOTAL_MIN, min);
            int hi = Math.Min(TOTAL_MAX, max);
            if (lo > hi) { lo = TOTAL_MIN; hi = TOTAL_MAX; } // 防呆：JSON 給反了
            TotalOptions.Clear();
            for (int i = lo; i <= hi; i++) TotalOptions.Add(i);
        }

        private void EnsureTotalOptionsCover(int uiTotal)
        {
            if (TotalOptions.Count == 0 ||
                uiTotal < TotalOptions[0] ||
                uiTotal > TotalOptions[TotalOptions.Count - 1])
            {
                BuildTotalOptions(TOTAL_MIN, TOTAL_MAX);
            }
        }

        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, TOTAL_MIN, TOTAL_MAX);

            // 增
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            // 減
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

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
        // 內部型別（C# 7.3 風格）
        // =====================================================================
        public sealed class OrderSlot : ViewModelBase
        {
            private int _displayNo;
            private int _selectedIndex; // 寫進 ORD 的實際 index（0..63）

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

    // === 若你沒有 RegMap.Read8Safe，可移除這段，並在上面呼叫改回 RegMap.Read8 ===
    internal static class RegMap
    {
        public static byte Read8(string name)
        {
            // 你的實作
            return 0;
        }
        public static byte Read8Safe(string name)
        {
            try { return Read8(name); } catch { return 0; }
        }
        public static void Write8(string name, byte value)
        {
            // 你的實作
        }
    }
}