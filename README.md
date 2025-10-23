using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class BWLoopVM : ViewModelBase
    {
        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _defaultValue = 0x01E;

        private string _lowAddr;
        private string _highAddr;
        private int _mask = 0x0C; // bit[3:2]
        private int _shift = 2;

        // D2 允許的最大值（0..2）
        private int _d2Max = 2;

        public ObservableCollection<int> D2Options { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> D1Options { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> D0Options { get; } = new ObservableCollection<int>();

        public int FCNT2_D2 { get { return GetValue<int>(); } set { SetValue(ClampD2(value)); } }
        public int FCNT2_D1 { get { return GetValue<int>(); } set { SetValue(ClampNibble(value)); } }
        public int FCNT2_D0 { get { return GetValue<int>(); } set { SetValue(ClampNibble(value)); } }

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
            SetFromValue(_defaultValue);
        }

        public void LoadFrom(JObject bwLoopControlNode)
        {
            if (bwLoopControlNode == null) return;

            var fcnt2 = bwLoopControlNode["fcnt2"] as JObject;
            if (fcnt2 == null) return;

            _defaultValue = TryParseInt((string)fcnt2["default"], _defaultValue);
            _min          = TryParseInt((string)fcnt2["min"],     _min);
            _max          = TryParseInt((string)fcnt2["max"],     _max);

            // 允許 JSON 覆蓋 D2 上限
            _d2Max = (int?)fcnt2["d2Max"] ?? _d2Max;
            if (_d2Max < 0) _d2Max = 0;
            if (_d2Max > 15) _d2Max = 15;

            var targets = fcnt2["targets"] as JObject;
            if (targets != null)
            {
                _lowAddr  = (string)targets["low"];
                _highAddr = (string)targets["high"];
                _mask     = TryParseInt((string)targets["mask"], _mask);
                _shift    = (int?)targets["shift"] ?? _shift;
            }

            BuildNibbleOptions(); // 重建 0.._d2Max
            SetFromValue(Clamp(_defaultValue, _min, _max));
        }

        public void ResetToDefault() => SetFromValue(Clamp(_defaultValue, _min, _max));

        private void Apply()
        {
            if (string.IsNullOrEmpty(_lowAddr) || string.IsNullOrEmpty(_highAddr))
                return;

            int  value    = FCNT2_Value;
            byte lowByte  = (byte)(value & 0xFF);
            byte hiBits   = (byte)((value >> 8) & 0x03);

            RegMap.Write8(_lowAddr, lowByte);

            byte cur  = RegMap.Read8(_highAddr);
            byte next = (byte)((cur & ~(byte)(_mask & 0xFF)) | (byte)(((hiBits << _shift) & _mask) & 0xFF));
            RegMap.Write8(_highAddr, next);
        }

        private void BuildNibbleOptions()
        {
            D2Options.Clear(); D1Options.Clear(); D0Options.Clear();
            for (int i = 0; i <= _d2Max; i++) D2Options.Add(i);
            for (int i = 0; i <= 15; i++) { D1Options.Add(i); D0Options.Add(i); }
        }

        private void SetFromValue(int value)
        {
            value = Clamp(value, _min, _max);
            FCNT2_D2 = ClampD2((value >> 8) & 0xF);
            FCNT2_D1 = (value >> 4) & 0xF;
            FCNT2_D0 = (value >> 0) & 0xF;
        }

        private static int Clamp(int v, int min, int max) { if (v < min) return min; if (v > max) return max; return v; }
        private static int ClampNibble(int v) { if (v < 0) return 0; if (v > 15) return 15; return v; }
        private int ClampD2(int v) { if (v < 0) return 0; if (v > _d2Max) return _d2Max; return v; }

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