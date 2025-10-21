using System;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // === JSON targets/obj ===
        private string _totalTarget;
        private string[] _ordTargets = new string[0];
        private string _triggerTarget; private byte _triggerValue;

        private string _f1Low, _f1High; private byte _f1Mask; private int _f1Shift;
        private string _f2Low, _f2High; private byte _f2Mask; private int _f2Shift;

        // === UI: pattern index 可選清單（由 BistModeVM 填入） ===
        public ObservableCollection<int> PatternIndexOptions { get; } = new ObservableCollection<int>();

        // === Total 可選 1..22 ===
        public ObservableCollection<int> TotalOptions { get; } =
            new ObservableCollection<int>(Enumerable.Range(1, 22));

        public int Total
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                // 依 Total 動態調整 Orders 的長度
                while (Orders.Count < Total) Orders.Add(0);
                while (Orders.Count > Total) Orders.RemoveAt(Orders.Count - 1);
            }
        }

        // === ORD0..ORDn-1 ===
        public ObservableCollection<int> Orders { get; } = new ObservableCollection<int>();

        // === FCNT 整數（合成後寫入用） ===
        public int FCNT1 { get => GetValue<int>(); private set => SetValue(value); }
        public int FCNT2 { get => GetValue<int>(); private set => SetValue(value); }

        // === 三位 16 進位 digit 的選項 ===
        public ObservableCollection<int> FcntD2Options { get; } =
            new ObservableCollection<int>(Enumerable.Range(0, 3));   // 0..2
        public ObservableCollection<int> FcntD1Options { get; } =
            new ObservableCollection<int>(Enumerable.Range(0, 16));  // 0..F
        public ObservableCollection<int> FcntD0Options { get; } =
            new ObservableCollection<int>(Enumerable.Range(0, 16));  // 0..F

        // FCNT1 digits
        public int FCNT1_D2 { get => GetValue<int>(); set { if (SetValue(value)) FCNT1 = CombineHex3(FCNT1_D2, FCNT1_D1, FCNT1_D0); } }
        public int FCNT1_D1 { get => GetValue<int>(); set { if (SetValue(value)) FCNT1 = CombineHex3(FCNT1_D2, FCNT1_D1, FCNT1_D0); } }
        public int FCNT1_D0 { get => GetValue<int>(); set { if (SetValue(value)) FCNT1 = CombineHex3(FCNT1_D2, FCNT1_D1, FCNT1_D0); } }
        // FCNT2 digits
        public int FCNT2_D2 { get => GetValue<int>(); set { if (SetValue(value)) FCNT2 = CombineHex3(FCNT2_D2, FCNT2_D1, FCNT2_D0); } }
        public int FCNT2_D1 { get => GetValue<int>(); set { if (SetValue(value)) FCNT2 = CombineHex3(FCNT2_D2, FCNT2_D1, FCNT2_D0); } }
        public int FCNT2_D0 { get => GetValue<int>(); set { if (SetValue(value)) FCNT2 = CombineHex3(FCNT2_D2, FCNT2_D1, FCNT2_D0); } }

        // 顯示用
        public string Title { get; private set; } = "Auto Run";

        public System.Windows.Input.ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ===== 供 BistModeVM 呼叫載入 JSON 區塊 =====
        public void LoadFrom(JObject jo)
        {
            if (jo == null) return;
            Title = (string)jo["title"] ?? "Auto Run";

            // total
            var totalObj = jo["total"] as JObject;
            _totalTarget = (string)totalObj?["target"];
            int defTotal = totalObj?["default"]?.Value<int>() ?? 22;
            defTotal = Clamp(defTotal, TotalOptions.First(), TotalOptions.Last());
            Total = defTotal;

            // orders
            var orders = jo["orders"] as JObject;
            _ordTargets = orders?["targets"]?.ToObject<string[]>() ?? new string[0];
            var defs = orders?["defaults"]?.ToObject<int[]>() ?? new int[0];
            Orders.Clear();
            for (int i = 0; i < Total; i++)
                Orders.Add(i < defs.Length ? defs[i] : 0);

            // fcnt1
            var f1 = jo["fcnt1"] as JObject;
            ParseFcnt(f1, out var f1Default, out _f1Low, out _f1High, out _f1Mask, out _f1Shift);
            InitFcntDigits(f1Default, out var d2, out var d1, out var d0);
            FCNT1_D2 = d2; FCNT1_D1 = d1; FCNT1_D0 = d0; FCNT1 = f1Default;

            // fcnt2
            var f2 = jo["fcnt2"] as JObject;
            ParseFcnt(f2, out var f2Default, out _f2Low, out _f2High, out _f2Mask, out _f2Shift);
            InitFcntDigits(f2Default, out d2, out d1, out d0);
            FCNT2_D2 = d2; FCNT2_D1 = d1; FCNT2_D0 = d0; FCNT2 = f2Default;

            // trigger
            var trig = jo["trigger"] as JObject;
            _triggerTarget = (string)trig?["target"];
            _triggerValue  = ParseHexByte((string)trig?["value"], 0x3F);
        }

        // ===== Apply =====
        private void Apply()
        {
            // 1) Total
            if (!string.IsNullOrEmpty(_totalTarget))
                RegMap.Write8(_totalTarget, (byte)Total);

            // 2) ORD0..ORDn-1
            for (int i = 0; i < Total && i < _ordTargets.Length; i++)
                RegMap.Write8(_ordTargets[i], (byte)(Orders[i] & 0x3F));

            // 3) FCNT1 / 2
            WriteFcnt(_f1Low, _f1High, _f1Mask, _f1Shift, FCNT1);
            WriteFcnt(_f2Low, _f2High, _f2Mask, _f2Shift, FCNT2);

            // 4) Trigger
            if (!string.IsNullOrEmpty(_triggerTarget))
                RegMap.Write8(_triggerTarget, _triggerValue);
        }

        // ===== helpers =====
        private static int CombineHex3(int d2, int d1, int d0)
        {
            d2 = Clamp(d2, 0, 2);
            d1 = Clamp(d1, 0, 15);
            d0 = Clamp(d0, 0, 15);
            return ((d2 & 0x3) << 8) | ((d1 & 0xF) << 4) | (d0 & 0xF);
        }

        private static void InitFcntDigits(int value, out int d2, out int d1, out int d0)
        {
            d2 = (value >> 8) & 0x3;
            d1 = (value >> 4) & 0xF;
            d0 = (value      ) & 0xF;
        }

        private static void ParseFcnt(JObject o, out int def,
                                      out string low, out string high, out byte mask, out int shift)
        {
            def = ParseHexInt((string)o?["default"], 0);
            var t = o?["targets"] as JObject;
            low   = (string)t?["low"];
            high  = (string)t?["high"];
            mask  = ParseHexByte((string)t?["mask"], 0x00);
            shift = t?["shift"]?.Value<int>() ?? 0;
        }

        private static byte ParseHexByte(string s, byte fb)
        {
            if (string.IsNullOrWhiteSpace(s)) return fb;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
            if (byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var b)) return b;
            return fb;
        }
        private static int ParseHexInt(string s, int fb)
        {
            if (string.IsNullOrWhiteSpace(s)) return fb;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
            if (int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var v)) return v;
            return fb;
        }

        private static void WriteFcnt(string low, string high, byte mask, int shift, int value)
        {
            if (!string.IsNullOrEmpty(low))
                RegMap.Write8(low, (byte)(value & 0xFF));
            if (!string.IsNullOrEmpty(high) && mask != 0)
            {
                byte hi = (byte)(((value >> 8) & 0x03) << shift);
                RegMap.Rmw(high, mask, hi);
            }
        }

        private static int Clamp(int v, int min, int max)
        { if (v < min) return min; if (v > max) return max; return v; }
    }
}