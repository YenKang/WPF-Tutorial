using Newtonsoft.Json.Linq;
using System.Collections.ObjectModel;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class BWLoopVM : ViewModelBase
    {
        private JObject _cfg;
        private bool _preferHw = false; // ✅ 硬體優先旗標

        // === JSON 參數 ===
        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _defaultValue = 0x01E;

        // === 暫存器目標 ===
        private string _lowAddr = "BIST_FCNT2_LO";   // 0x0043
        private string _highAddr = "BIST_FCNT2_HI";  // 0x0044
        private int _mask = 0x0C;                    // 高位 mask [3:2]
        private int _shift = 2;                      // 起始 bit2

        // === ComboBox 選項 ===
        public ObservableCollection<int> D2Options { get; } = new ObservableCollection<int>(new[] { 0, 1, 2, 3 });
        public ObservableCollection<int> D1Options { get; } = new ObservableCollection<int>(Generate(0, 15));
        public ObservableCollection<int> D0Options { get; } = new ObservableCollection<int>(Generate(0, 15));

        // === 三個 nibble ===
        public int FCNT2_D2 { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }
        public int FCNT2_D1 { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }
        public int FCNT2_D0 { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }

        // === 合併為 10-bit ===
        public int FCNT2_Value
        {
            get
            {
                int v = ((FCNT2_D2 & 0x3) << 8) | ((FCNT2_D1 & 0xF) << 4) | (FCNT2_D0 & 0xF);
                if (v < _min) v = _min;
                if (v > _max) v = _max;
                return v;
            }
        }

        public ICommand ApplyCommand { get; }

        public BWLoopVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
            BuildNibbleOptions();
            SetFromValue(_defaultValue);
        }

        // === 初始化：讀取 JSON 並建構 UI ===
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            if (_cfg["default"] != null)
                _defaultValue = TryParseInt(_cfg["default"].ToString(), _defaultValue);

            SetFromValue(_defaultValue);

            // ✅ 若已執行過 Set，則以硬體真值覆蓋 UI
            if (_preferHw) TryRefreshFromRegister();
        }

        // === 寫入 ===
        private void Apply()
        {
            try
            {
                int value = FCNT2_Value;

                // 寫低位
                byte low = (byte)(value & 0xFF);
                RegMap.Write8(_lowAddr, low);

                // 寫高位 (2bits)
                byte hiBits = (byte)((value >> 8) & 0x03);
                byte cur = RegMap.Read8(_highAddr);
                byte next = (byte)((cur & ~_mask) | ((hiBits << _shift) & _mask));
                RegMap.Write8(_highAddr, next);

                System.Diagnostics.Debug.WriteLine($"[BWLoop] Apply success: 0x{value:X3}");
                _preferHw = true;              // ✅ 寫完後，改為硬體優先模式
                TryRefreshFromRegister();      // ✅ 寫完立刻回讀同步
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[BWLoop] Apply failed: " + ex.Message);
            }
        }

        // === 回讀暫存器 ===
        public void RefreshFromRegister()
        {
            try
            {
                int low = RegMap.Read8(_lowAddr) & 0xFF;
                int high = RegMap.Read8(_highAddr) & 0xFF;
                int hi2 = (high & _mask) >> _shift;
                int v10 = ((hi2 << 8) | low) & 0x3FF;

                FCNT2_D2 = (v10 >> 8) & 0x03;
                FCNT2_D1 = (v10 >> 4) & 0x0F;
                FCNT2_D0 = (v10 >> 0) & 0x0F;

                System.Diagnostics.Debug.WriteLine($"[BWLoop] Refresh OK: {v10:X3} (D2={FCNT2_D2}, D1={FCNT2_D1}, D0={FCNT2_D0})");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[BWLoop] Refresh failed: " + ex.Message);
            }
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch { return false; }
        }

        // === 小工具 ===
        private static int ClampNibble(int v) => v < 0 ? 0 : (v > 15 ? 15 : v);

        private static void SetFromValue(int value)
        {
            value = value & 0x3FF;
            int d2 = (value >> 8) & 0x03;
            int d1 = (value >> 4) & 0x0F;
            int d0 = (value >> 0) & 0x0F;

            // 通常在 ViewModelBase 裡有 SetValue 機制
        }

        private static int TryParseInt(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            if (s.StartsWith("0x"))
                return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out int hex) ? hex : fallback;
            return int.TryParse(s, out int v) ? v : fallback;
        }

        private static int[] Generate(int start, int end)
        {
            var arr = new int[end - start + 1];
            for (int i = 0; i < arr.Length; i++) arr[i] = start + i;
            return arr;
        }

        private void BuildNibbleOptions()
        {
            D2Options.Clear();
            for (int i = 0; i <= 3; i++) D2Options.Add(i);
            D1Options.Clear();
            D0Options.Clear();
            for (int i = 0; i <= 15; i++) { D1Options.Add(i); D0Options.Add(i); }
        }
    }
}