using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class BWLoopVM : ViewModelBase
    {
        // JSON 參數
        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _defaultValue = 0x01E;

        // targets
        private string _lowAddr;   // BIST_FCNT2_LO (0043h)
        private string _highAddr;  // BIST_FCNT2_HI (0044h)
        private int _mask = 0x0C;  // 高位遮罩 -> bit[3:2]
        private int _shift = 2;    // 寫入位移 -> 從 bit2 起

        // 0..F 選項
        public ObservableCollection<int> D2Options { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> D1Options { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> D0Options { get; } = new ObservableCollection<int>();

        // 三個 nibble
        public int FCNT2_D2 { get { return GetValue<int>(); } set { SetValue(ClampNibble(value)); } }
        public int FCNT2_D1 { get { return GetValue<int>(); } set { SetValue(ClampNibble(value)); } }
        public int FCNT2_D0 { get { return GetValue<int>(); } set { SetValue(ClampNibble(value)); } }

        // 合成值 (0..0x3FF)
        public int FCNT2_Value
        {
            get
            {
                int v = ((FCNT2_D2 & 0xF) << 8) | ((FCNT2_D1 & 0xF) << 4) | (FCNT2_D0 & 0xF);
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
            SetFromValue(_defaultValue); // JSON 未載入前的保底
        }

        /// 從 bwLoopControl 載入（吃到 fcnt2/targets 結構）
        public void LoadFrom(JObject bwLoopControlNode)
        {
            if (bwLoopControlNode == null) return;

            var fcnt2 = bwLoopControlNode["fcnt2"] as JObject;
            if (fcnt2 == null) return;

            _defaultValue = TryParseInt((string)fcnt2["default"], _defaultValue);
            _min          = TryParseInt((string)fcnt2["min"],     _min);
            _max          = TryParseInt((string)fcnt2["max"],     _max);

            var targets = fcnt2["targets"] as JObject;
            if (targets != null)
            {
                _lowAddr  = (string)targets["low"];
                _highAddr = (string)targets["high"];
                _mask     = TryParseInt((string)targets["mask"], _mask);
                _shift    = (int?)targets["shift"] ?? _shift;
            }

            SetFromValue(Clamp(_defaultValue, _min, _max));
        }

        public void ResetToDefault() => SetFromValue(Clamp(_defaultValue, _min, _max));

        private void Apply()
        {
            if (string.IsNullOrEmpty(_lowAddr) || string.IsNullOrEmpty(_highAddr))
                return;

            int  value    = FCNT2_Value;              // 0..0x3FF
            byte lowByte  = (byte)(value & 0xFF);     // 低 8 位
            byte hiBits   = (byte)((value >> 8) & 0x03); // 高 2 位 (FCNT2[9:8])

            // 寫低位
            RegMap.Write8(_lowAddr, lowByte);

            // 高位 RMW：把兩個 bits 放進 high 的 bit[3:2]
            byte cur  = RegMap.Read8(_highAddr);
            byte next = (byte)((cur & ~(byte)(_mask & 0xFF)) | (byte)(((hiBits << _shift) & _mask) & 0xFF));
            RegMap.Write8(_highAddr, next);
        }

        // Helpers
        private void BuildNibbleOptions()
        {
            D2Options.Clear(); D1Options.Clear(); D0Options.Clear();
            for (int i = 0; i <= 15; i++) { D2Options.Add(i); D1Options.Add(i); D0Options.Add(i); }
        }

        private void SetFromValue(int value)
        {
            value = Clamp(value, _min, _max);
            FCNT2_D2 = (value >> 8) & 0xF;
            FCNT2_D1 = (value >> 4) & 0xF;
            FCNT2_D0 = (value >> 0) & 0xF;
        }

        private static int Clamp(int v, int min, int max) { if (v < min) return min; if (v > max) return max; return v; }
        private static int ClampNibble(int v) { if (v < 0) return 0; if (v > 15) return 15; return v; }

        private static int TryParseInt(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out var hv) ? hv : fallback;
            return int.TryParse(s, out var dv) ? dv : fallback;
        }
    }
}