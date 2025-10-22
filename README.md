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

        public ICommand ApplyCommand { get; private set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
            HRes = 1920; VRes = 720; M = 5; N = 7;
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
            byte h_high  = (byte)((th >> 8) & 0x1F);  // 高位 5bit
            byte h_plus1 = (byte)((hRes - th * m) == 0 ? 0 : 1);

            byte v_low   = (byte)(tv & 0xFF);
            byte v_high  = (byte)((tv >> 8) & 0x0F);  // 高位 4bit
            byte v_plus1 = (byte)((vRes - tv * n) == 0 ? 0 : 1);

            // 🧩 寫入對應暫存器
            // 1. 水平 (003C, 003D, 0040)
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", h_low);

            byte cur003D = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H");
            byte next003D = (byte)((cur003D & 0xE0) | (h_high & 0x1F));
            if (next003D != cur003D)
                RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_H", next003D);

            RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", h_plus1);

            // 2. 垂直 (003E, 003F, 0041)
            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", v_low);

            byte cur003F = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H");
            byte next003F = (byte)((cur003F & 0xF0) | (v_high & 0x0F));
            if (next003F != cur003F)
                RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_H", next003F);

            RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", v_plus1);

            // 🔧 Debug 紀錄
            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] H=0x{th:X4} ({hRes}/{m}), V=0x{tv:X4} ({vRes}/{n}), " +
                $"H_P1={h_plus1}, V_P1={v_plus1}"
            );

            _dirty = false;
        }
    }
}