using System;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;

        private bool _inInit = false;
        private bool _dirty  = false;
        public bool CanApply { get { return _dirty; } }

        private int _mMin = 2, _mMax = 40;
        private int _nMin = 2, _nMax = 40;

        private const byte PATTERN_INDEX_CHESSBOARD = 0x0A;

        public int HRes { get { return GetValue<int>(); } set { SetValue(value); } }
        public int VRes { get { return GetValue<int>(); } set { SetValue(value); } }

        public int M
        {
            get { return GetValue<int>(); }
            set
            {
                if (value < _mMin) value = _mMin;
                if (value > _mMax) value = _mMax;
                SetValue(value);
                if (!_inInit) _dirty = true;
            }
        }

        public int N
        {
            get { return GetValue<int>(); }
            set
            {
                if (value < _nMin) value = _nMin;
                if (value > _nMax) value = _nMax;
                SetValue(value);
                if (!_inInit) _dirty = true;
            }
        }

        // 衍生值 (純計算)
        public int ToggleH { get { return (M > 0) ? (HRes / M) : 0; } }
        public int ToggleV { get { return (N > 0) ? (VRes / N) : 0; } }
        public int RemainderH { get { return HRes - (ToggleH * M); } }
        public int RemainderV { get { return VRes - (ToggleV * N); } }
        public int H_Plus1 { get { return (RemainderH == 0) ? 0 : 1; } }
        public int V_Plus1 { get { return (RemainderV == 0) ? 0 : 1; } }

        public ICommand ApplyCommand { get; private set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
            HRes = 1920; 
            VRes = 720; 
            M = 5; 
            N = 7;
        }

        public void LoadFrom(JObject node)
        {
            _inInit = true;
            try
            {
                _cfg = node;
                var fields = node["fields"] as JArray;
                if (fields != null)
                {
                    foreach (JObject f in fields)
                    {
                        string key = (string)f["key"];
                        int def = (int?)f["default"] ?? 0;
                        int min = (int?)f["min"] ?? 0;
                        int max = (int?)f["max"] ?? 0;

                        switch (key)
                        {
                            case "H_RES": HRes = def; break;
                            case "V_RES": VRes = def; break;
                            case "M":
                                _mMin = (min > 0) ? min : _mMin;
                                _mMax = (max > 0) ? max : _mMax;
                                M = def;
                                break;
                            case "N":
                                _nMin = (min > 0) ? min : _nMin;
                                _nMax = (max > 0) ? max : _nMax;
                                N = def;
                                break;
                        }
                    }
                }
            }
            finally
            {
                _inInit = false;
                _dirty  = false;
            }
        }

        private void Apply()
        {
            if (_cfg == null) return;

            int hRes = HRes, vRes = VRes, m = M, n = N;
            int th = (m > 0) ? hRes / m : 0;
            int tv = (n > 0) ? vRes / n : 0;

            th = Math.Clamp(th, 0, 0x1FFF);
            tv = Math.Clamp(tv, 0, 0x0FFF);

            byte h_low   = (byte)(th & 0xFF);
            byte h_high  = (byte)((th >> 8) & 0xFF);
            byte h_plus1 = (byte)((hRes - th * m) == 0 ? 0 : 1);

            byte v_low   = (byte)(tv & 0xFF);
            byte v_high  = (byte)((tv >> 8) & 0xFF);
            byte v_plus1 = (byte)((vRes - tv * n) == 0 ? 0 : 1);

            // 僅作為演算示例，無任何寫入
            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] H={th} (0x{th:X}) L={h_low:X2} H={h_high:X2} P1={h_plus1}, " +
                $"V={tv} (0x{tv:X}) L={v_low:X2} H={v_high:X2} P1={v_plus1}"
            );

            _dirty = false;
        }
    }
}