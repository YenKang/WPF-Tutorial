using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Globalization;
using System.Windows.Input;
using Newtonsoft.Json.Linq;

namespace BistMode.ViewModels
{
    public sealed class FLKRGBVM : ViewModelBase
    {
        public ObservableCollection<PlaneVM> Planes { get; } =
            new ObservableCollection<PlaneVM>
            {
                new PlaneVM(1), new PlaneVM(2), new PlaneVM(3), new PlaneVM(4)
            };

        public ICommand ApplyCommand { get; }

        private JObject _cfg;                       // 整個 flickerMatrixControl 區塊
        private readonly List<PairWrite> _pairs = new List<PairWrite>(); // pairs 陣列

        public FLKRGBVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        // 由 PatternItem.FlickerMatrixControl 丟進來（object/JObject 都可）
        public void LoadFrom(object jsonCfg)
        {
            _pairs.Clear();
            if (jsonCfg is JObject jo) _cfg = jo;
            else if (jsonCfg != null) _cfg = JObject.FromObject(jsonCfg);
            else { _cfg = null; return; }

            // defaults
            var d = _cfg["defaults"] as JObject;
            if (d != null)
            {
                ApplyDefault(d["P1"] as JObject, Planes[0]);
                ApplyDefault(d["P2"] as JObject, Planes[1]);
                ApplyDefault(d["P3"] as JObject, Planes[2]);
                ApplyDefault(d["P4"] as JObject, Planes[3]);
            }

            // pairs
            var arr = _cfg["pairs"] as JArray;
            if (arr != null)
            {
                foreach (var it in arr)
                {
                    var p = it as JObject; if (p == null) continue;
                    var target = (string)p["target"]; if (string.IsNullOrWhiteSpace(target)) continue;
                    var low    = (string)p["low"];    if (string.IsNullOrWhiteSpace(low))    continue;
                    var high   = (string)p["high"];   if (string.IsNullOrWhiteSpace(high))   continue;

                    _pairs.Add(new PairWrite { Target = target, LowSrc = low, HighSrc = high });
                }
            }
        }

        private static void ApplyDefault(JObject jo, PlaneVM vm)
        {
            if (jo == null || vm == null) return;
            vm.R = Clamp4(GetOr(jo, "R", 0));
            vm.G = Clamp4(GetOr(jo, "G", 0));
            vm.B = Clamp4(GetOr(jo, "B", 0));
        }

        private static int GetOr(JObject o, string key, int fb)
        {
            if (o == null) return fb;
            if (!o.TryGetValue(key, StringComparison.OrdinalIgnoreCase, out var t)) return fb;
            return TryParseInt(t.ToString(), out var v) ? v : fb;
        }

        private static bool TryParseInt(string s, out int v)
        {
            v = 0; if (string.IsNullOrWhiteSpace(s)) return false;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
            return int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v);
        }

        private static int  Clamp4(int x) => x < 0 ? 0 : (x > 15 ? 15 : x);
        private static byte Nib(int x)    => (byte)(Clamp4(x) & 0x0F);
        private static byte PackTwo4(int low, int high) => (byte)((Nib(high) << 4) | Nib(low));

        private void ExecuteWrite()
        {
            // 依 JSON pairs 逐筆打包並 Write8
            foreach (var p in _pairs)
            {
                int lo = ResolveSrc(p.LowSrc);
                int hi = ResolveSrc(p.HighSrc);
                byte val = PackTwo4(lo, hi);
                RegMap.Write8(p.Target, val);
                System.Diagnostics.Debug.WriteLine($"[FLK] {p.Target} <= {val:X2} ({p.LowSrc},{p.HighSrc})");
            }
        }

        // "P2.R" -> 取對應的 plane與channel 值
        private int ResolveSrc(string src)
        {
            if (string.IsNullOrWhiteSpace(src)) return 0;
            var s = src.Trim().ToUpperInvariant(); // P1.R / P3.B ...

            int plane = 1;
            if (s.Length >= 2 && s[0] == 'P' && char.IsDigit(s[1])) plane = s[1] - '0';
            if (plane < 1 || plane > 4) plane = 1;

            var vm = Planes[plane - 1];
            if (s.EndsWith(".R")) return vm.R;
            if (s.EndsWith(".G")) return vm.G;
            if (s.EndsWith(".B")) return vm.B;
            return 0;
        }

        private sealed class PairWrite
        {
            public string Target;  // e.g. "BIST_FLK_P1P2_R"
            public string LowSrc;  // "P1.R"
            public string HighSrc; // "P2.R"
        }
    }

    public sealed class PlaneVM : ViewModelBase
    {
        public int Index { get; }
        public ObservableCollection<int> LevelOptions { get; } =
            new ObservableCollection<int>(Generate(0, 15));

        public int R { get => GetValue<int>(); set => SetValue(value); }
        public int G { get => GetValue<int>(); set => SetValue(value); }
        public int B { get => GetValue<int>(); set => SetValue(value); }

        public PlaneVM(int idx) { Index = idx; R = 0; G = 0; B = 0; }

        private static int[] Generate(int a, int b)
        {
            var n = b - a + 1; var arr = new int[n];
            for (int i = 0; i < n; i++) arr[i] = a + i;
            return arr;
        }
    }
}