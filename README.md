using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class BWLoopVM : ViewModelBase
    {
        private JObject _cfg;

        // 一旦 Set 成功 -> 後續以硬體真值覆蓋 UI
        private bool _preferHw = false;

        // ===== JSON: 整值範圍與預設 =====
        private int _min = 0x000;
        private int _max = 0x3FF;          // 10-bit
        private int _defaultValue = 0x01E; // 先給既有預設, 會被 JSON 覆蓋

        // ===== 寫入目標（可由 JSON targets 覆蓋）=====
        private string _lowAddr  = "BIST_FCNT2_LO";  // 0x0043
        private string _highAddr = "BIST_FCNT2_HI";  // 0x0044
        private byte   _mask     = 0x0C;             // 高位 2bits 的 mask (bit[3:2])
        private int    _shift    = 2;                // 從 bit2 起

        // ===== UI 選項 =====
        public ObservableCollection<int> D2Options { get; } = new ObservableCollection<int>(new[] { 0, 1, 2, 3 });
        public ObservableCollection<int> D1Options { get; } = new ObservableCollection<int>(Generate(0, 15));
        public ObservableCollection<int> D0Options { get; } = new ObservableCollection<int>(Generate(0, 15));

        // ===== 三個 nibble（D2 實際只用 2bits，UI 嚴格限制 0..3）=====
        public int FCNT2_D2 { get => GetValue<int>(); set => SetValue(value < 0 ? 0 : (value > 3 ? 3 : value)); }
        public int FCNT2_D1 { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }
        public int FCNT2_D0 { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }

        // ===== 合併為 10-bit =====
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
            // 預先塞選項（保留你原本風格）
            if (D1Options.Count == 0 || D0Options.Count == 0)
            {
                D1Options.Clear(); D0Options.Clear();
                for (int i = 0; i <= 15; i++) { D1Options.Add(i); D0Options.Add(i); }
            }
            // 初始 UI 值（避免空白）
            SetFromValue(_defaultValue);
        }

        // ===== 從 JSON 載入 =====
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 讀 fcnt2 節點（default/min/max）
            if (_cfg["fcnt2"] is JObject fcnt2)
            {
                _defaultValue = TryParseInt((string)fcnt2["default"], _defaultValue);
                _min          = TryParseInt((string)fcnt2["min"],      _min);
                _max          = TryParseInt((string)fcnt2["max"],      _max);
            }

            // 讀 targets（low/high/mask/shift）
            if (_cfg["targets"] is JObject tgt)
            {
                _lowAddr  = (string)tgt["low"]  ?? _lowAddr;
                _highAddr = (string)tgt["high"] ?? _highAddr;
                _mask     = ParseHexByte((string)tgt["mask"], _mask);
                _shift    = (int?)tgt["shift"] ?? _shift;
            }

            // 先用 JSON default 展開到 D2/D1/D0
            SetFromValue(Clamp(_defaultValue, _min, _max));

            // 若曾 Apply 過 -> 以硬體真值覆蓋
            if (_preferHw) TryRefreshFromRegister();

            System.Diagnostics.Debug.WriteLine(
                $"[BWLoop] LoadFrom: def=0x{_defaultValue:X3}, D2={FCNT2_D2},D1={FCNT2_D1},D0={FCNT2_D0}, preferHw={_preferHw}");
        }

        // ===== Set（永遠可寫；寫完切硬體優先＋回讀）=====
        private void Apply()
        {
            bool ok = false;

            try
            {
                int value = FCNT2_Value;

                // 寫低 8 位
                byte low = (byte)(value & 0xFF);
                RegMap.Write8(_lowAddr, low);
                ok = true;

                // 寫高 2 位（RMW）
                byte hiBits  = (byte)((value >> 8) & 0x03);
                byte curHigh = RegMap.Read8(_highAddr);
                byte next    = (byte)((curHigh & ~_mask) | (((hiBits << _shift) & _mask)));
                RegMap.Write8(_highAddr, next);
                ok = true;

                System.Diagnostics.Debug.WriteLine($"[BWLoop] Apply OK -> 0x{value:X3}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[BWLoop] Apply failed: " + ex.Message);
            }

            if (ok)
            {
                _preferHw = true;     // 之後以硬體真值為主
                TryRefreshFromRegister(); // 寫完立刻回讀同步 UI
            }
        }

        // ===== 回讀暫存器，拆回 D2/D1/D0 =====
        public void RefreshFromRegister()
        {
            int low  = RegMap.Read8(_lowAddr)  & 0xFF;
            int high = RegMap.Read8(_highAddr) & 0xFF;

            int hi2  = (high & _mask) >> _shift;      // 取出高位 2bits
            int v10  = ((hi2 & 0x03) << 8) | low;     // 合成 10-bit

            // 展開回 UI（D2 嚴格 0..3）
            FCNT2_D2 = ((v10 >> 8) & 0x0F) > 3 ? 3 : ((v10 >> 8) & 0x0F);
            FCNT2_D1 = (v10 >> 4) & 0x0F;
            FCNT2_D0 =  v10       & 0x0F;

            System.Diagnostics.Debug.WriteLine($"[BWLoop] Refresh OK: low=0x{low:X2}, high=0x{high:X2} -> 0x{v10:X3}");
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[BWLoop] Refresh skipped: " + ex.Message);
                return false;
            }
        }

        // ===== Helpers =====
        private static int Clamp(int v, int min, int max) => (v < min) ? min : (v > max ? max : v);
        private static int ClampNibble(int v) => v < 0 ? 0 : (v > 15 ? 15 : v);

        private void SetFromValue(int value)
        {
            value &= 0x3FF; // 只保留 10-bit

            int d2 = (value >> 8) & 0x0F; // 與你舊版一致（但 UI 會再限制 0..3）
            int d1 = (value >> 4) & 0x0F;
            int d0 = (value >> 0) & 0x0F;

            FCNT2_D2 = d2 > 3 ? 3 : d2;   // 嚴格 2bits
            FCNT2_D1 = d1;
            FCNT2_D0 = d0;
        }

        private static int TryParseInt(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out int vHex) ? vHex : fallback;
            return int.TryParse(s, out int v) ? v : fallback;
        }

        private static byte ParseHexByte(string hexOrNull, byte fallback)
        {
            if (string.IsNullOrWhiteSpace(hexOrNull)) return fallback;
            string s = hexOrNull.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
            return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out byte b) ? b : fallback;
        }

        private static int[] Generate(int a, int b)
        {
            int n = b - a + 1;
            var arr = new int[n];
            for (int i = 0; i < n; i++) arr[i] = a + i;
            return arr;
        }
    }
}