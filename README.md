using System;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;

        // 動態上下限（吃 JSON）
        private int _mMin = 2, _mMax = 40;
        private int _nMin = 2, _nMax = 40;

        // 快取 JSON 區塊
        private JArray _writesH;
        private JArray _writesHPlus1;
        private JArray _writesV;
        private JArray _writesVPlus1;
        private JObject _trigger;

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
                RaiseComputed();
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
                RaiseComputed();
            }
        }

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
            HRes = 1920; VRes = 720; M = 5; N = 7;
        }

        public void LoadFrom(JObject patternNode)
        {
            _cfg = patternNode;

            // fields
            var fields = patternNode["fields"] as JArray;
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

            // writes 區塊（跟 GrayLevel/AutoRun 一樣用單一 JArray 交給 ExecuteRow）
            _writesH       = (patternNode["writesH"]?["writes"] as JArray) ?? new JArray();
            _writesHPlus1  = (patternNode["writesHPlus1"]?["writes"] as JArray) ?? new JArray();
            _writesV       = (patternNode["writesV"]?["writes"] as JArray) ?? new JArray();
            _writesVPlus1  = (patternNode["writesVPlus1"]?["writes"] as JArray) ?? new JArray();

            // trigger（學 AutoRun）
            _trigger = patternNode["patternSelect"]?["trigger"] as JObject;

            RaiseComputed();
        }

        private void Apply()
        {
            if (_cfg == null)
            {
                System.Windows.MessageBox.Show("❌ Chessboard 未載入 JSON。");
                return;
            }

            // 1) H：low/high & plus1
            int th = ToggleH;
            byte h_low   = (byte)(th & 0xFF);
            byte h_high  = (byte)((th >> 8) & 0xFF);
            byte h_plus1 = (byte)((RemainderH == 0) ? 0 : 1);

            ExecuteRow(_writesH,      0, 0, 0, h_low,   h_high);   // 003C/003D
            ExecuteRow(_writesHPlus1, 0, 0, 0, h_plus1, 0);        // 0040

            // 2) V：low/high & plus1
            int tv = ToggleV;
            byte v_low   = (byte)(tv & 0xFF);
            byte v_high  = (byte)((tv >> 8) & 0xFF);
            byte v_plus1 = (byte)((RemainderV == 0) ? 0 : 1);

            ExecuteRow(_writesV,      0, 0, 0, v_low,   v_high);   // 003E/003F
            ExecuteRow(_writesVPlus1, 0, 0, 0, v_plus1, 0);        // 0041

            // 3) Trigger：patternSelect → BIST_PT = 0x0A
            if (_trigger != null)
            {
                string on = (string)_trigger["on"] ?? "Apply";
                int delay = (int?)_trigger["delay_ms"] ?? 0;
                var writes = _trigger["writes"] as JArray;

                if (on == "Apply" && writes != null)
                {
                    if (delay > 0) System.Threading.Thread.Sleep(delay);
                    ExecuteRow(writes, 0, 0, 0, PATTERN_INDEX_CHESSBOARD, 0);
                }
            }

            System.Windows.MessageBox.Show("✅ Chessboard applied.");
        }

        // ====== 這個就是你們 GrayLevel/AutoRun 既有的執行介面 ======
        // 內部會用 foreach(writes) + ResolveSrc(D2/D1/D0/lowByte/highByte) 去跑
        private void ExecuteRow(JArray writes, byte d2, byte d1, byte d0, byte low, byte high)
        {
            if (writes == null) return;
            foreach (JObject w in writes)
            {
                string mode   = (string)w["mode"] ?? "write8";
                string target = (string)w["target"];
                string src    = (string)w["src"];

                int srcVal = ResolveSrc(src, d2, d1, d0, low, high);

                if (string.Equals(mode, "write8", StringComparison.OrdinalIgnoreCase))
                {
                    RegMap.Write8(target, (byte)(srcVal & 0xFF));
                }
                else if (string.Equals(mode, "rmw", StringComparison.OrdinalIgnoreCase))
                {
                    int mask  = ParseInt((string)w["mask"]);
                    int shift = (int?)(w["shift"]) ?? 0;

                    byte cur  = RegMap.Read8(target);
                    byte next = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
                    RegMap.Write8(target, next);
                }
            }
        }

        private static int ResolveSrc(string src, int d2, int d1, int d0, byte low, byte high)
        {
            if (string.IsNullOrEmpty(src)) return 0;
            switch (src)
            {
                case "D2": return d2;
                case "D1": return d1;
                case "D0": return d0;
                case "lowByte":  return low;
                case "highByte": return high;
                default:
                    if (TryParseInt(src, out int v)) return v;
                    return 0;
            }
        }

        private static bool TryParseInt(string s, out int v)
        {
            v = 0;
            if (string.IsNullOrWhiteSpace(s)) return false;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out v);
            return int.TryParse(s, out v);
        }
        private static int ParseInt(string s) => TryParseInt(s, out var v) ? v : 0;

        private void RaiseComputed()
        {
            RaisePropertyChanged(nameof(ToggleH));
            RaisePropertyChanged(nameof(ToggleV));
            RaisePropertyChanged(nameof(RemainderH));
            RaisePropertyChanged(nameof(RemainderV));
            RaisePropertyChanged(nameof(H_Plus1));
            RaisePropertyChanged(nameof(V_Plus1));
        }
    }
}