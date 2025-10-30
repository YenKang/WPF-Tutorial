using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Globalization;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    /// <summary>
    /// Flicker RGB Control
    /// - JSON:
    ///   { "defaults": { "P1": {"R":..,"G":..,"B":..}, ... },
    ///     "pairs": [ { "target":"REG", "low":"P1.R", "high":"P2.R" }, ... ] }
    /// - 寫入：每個 pair 將兩個 4bit (low/high) 打成 1 byte 寫入暫存器
    /// - 回讀：依 pairs 反解 nibble → 反填到 P1~P4 的 R/G/B
    /// - _preferHw：第一次 Set 成功後，後續切圖一律以硬體真值覆蓋 UI
    /// </summary>
    public sealed class FlickerRGBVM : ViewModelBase
    {
        // ---------- UI ----------
        public ObservableCollection<PlaneVM> Planes { get; } =
            new ObservableCollection<PlaneVM> { new PlaneVM(1), new PlaneVM(2), new PlaneVM(3), new PlaneVM(4) };

        public ICommand ApplyCommand { get; }

        // ---------- State ----------
        private JObject _cfg;
        private readonly List<PairWrite> _pairs = new List<PairWrite>();
        private bool _preferHw = false; // ✅ 寫入成功後 → 後續顯示以硬體真值為主

        public FlickerRGBVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ---------- Load JSON profile ----------
        public void LoadFrom(object jsonCfg)
        {
            _pairs.Clear();

            if (jsonCfg is JObject jo) _cfg = jo;
            else if (jsonCfg != null) _cfg = JObject.FromObject(jsonCfg);
            else { _cfg = null; return; }

            // 1) defaults: P1~P4 的 R/G/B
            if (_cfg["defaults"] is JObject d)
            {
                ApplyDefault(d["P1"] as JObject, Planes[0]);
                ApplyDefault(d["P2"] as JObject, Planes[1]);
                ApplyDefault(d["P3"] as JObject, Planes[2]);
                ApplyDefault(d["P4"] as JObject, Planes[3]);
            }

            // 2) pairs: 每個 pair 指定一顆 target 暫存器與 low/high 來源
            if (_cfg["pairs"] is JArray arr)
            {
                foreach (var it in arr)
                {
                    if (it is not JObject p) continue;
                    string target = (string)p["target"];
                    string low    = (string)p["low"];
                    string high   = (string)p["high"];
                    if (string.IsNullOrWhiteSpace(target) ||
                        string.IsNullOrWhiteSpace(low)    ||
                        string.IsNullOrWhiteSpace(high)) continue;

                    _pairs.Add(new PairWrite { Target = target, LowSrc = low, HighSrc = high });
                }
            }

            // 3) 若曾經 Set 過 → 切回來直接用硬體真值覆蓋
            if (_preferHw) TryRefreshFromRegister();
        }

        // ---------- Set：依 pairs 寫入暫存器 ----------
        private void Apply()
        {
            bool ok = false;

            try
            {
                foreach (var p in _pairs)
                {
                    int lo = ResolveSrc(p.LowSrc);     // 0..15
                    int hi = ResolveSrc(p.HighSrc);    // 0..15
                    byte v = PackByte(lo, hi);         // (hi<<4) | lo

                    RegMap.Write8(p.Target, v);
                    ok = true;

                    System.Diagnostics.Debug.WriteLine($"[FLK] {p.Target} <= {v:X2}  ({p.LowSrc},{p.HighSrc})");
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[FLK] Apply failed: " + ex.Message);
            }

            if (ok)
            {
                _preferHw = true;           // ✅ 之後 UI 以硬體真值為主
                TryRefreshFromRegister();   // ✅ 寫完立即回讀同步
            }
        }

        // ---------- 回讀：依 pairs 反解暫存器 → 填回 UI ----------
        public void RefreshFromRegister()
        {
            foreach (var p in _pairs)
            {
                byte val = RegMap.Read8(p.Target);
                int  lo  =  val        & 0x0F;
                int  hi  = (val >> 4)  & 0x0F;

                AssignChannel(p.LowSrc,  lo); // e.g. "P1.R"
                AssignChannel(p.HighSrc, hi); // e.g. "P2.G"
            }

            System.Diagnostics.Debug.WriteLine("[FLK] RefreshFromRegister OK.");
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[FLK] Refresh skipped: " + ex.Message);
                return false;
            }
        }

        // ---------- JSON helpers ----------
        private static void ApplyDefault(JObject jo, PlaneVM vm)
        {
            if (jo == null || vm == null) return;
            vm.R = GetOr(jo, "R", vm.R);
            vm.G = GetOr(jo, "G", vm.G);
            vm.B = GetOr(jo, "B", vm.B);
        }

        private static int GetOr(JObject jo, string key, int fb)
        {
            if (jo == null) return fb;
            if (jo.TryGetValue(key, StringComparison.OrdinalIgnoreCase, out var tok))
            {
                if (int.TryParse(tok.ToString(), out int v)) return ClampNibble(v);
                if (TryParseHex(tok.ToString(), out int hx)) return ClampNibble(hx);
            }
            return fb;
        }

        private static bool TryParseHex(string s, out int v)
        {
            v = 0;
            if (string.IsNullOrWhiteSpace(s)) return false;
            s = s.Trim();
            if (!s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) return false;
            return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
        }

        // ---------- 來源解析 / 反向指定 ----------
        /// <summary>將 "P1.R"/"P2.G"/"P3.B" 或常數字串解析為 0..15 的值。</summary>
        private int ResolveSrc(string src)
        {
            if (string.IsNullOrWhiteSpace(src)) return 0;
            src = src.Trim().ToUpperInvariant();

            // 常數: "5" / "0xA"
            if (int.TryParse(src, out int v) || TryParseHex(src, out v)) return ClampNibble(v);

            var plane = ResolvePlane(src);
            if (plane == null) return 0;

            if (src.EndsWith(".R")) return plane.R;
            if (src.EndsWith(".G")) return plane.G;
            if (src.EndsWith(".B")) return plane.B;

            return 0;
        }

        /// <summary>把 nibble 值寫回到 "P?.R/G/B" 對應的 UI 綁定屬性。</summary>
        private void AssignChannel(string dst, int nib)
        {
            if (string.IsNullOrWhiteSpace(dst)) return;
            dst = dst.Trim().ToUpperInvariant();
            nib = ClampNibble(nib);

            var plane = ResolvePlane(dst);
            if (plane == null) return;

            if      (dst.EndsWith(".R")) plane.R = nib;
            else if (dst.EndsWith(".G")) plane.G = nib;
            else if (dst.EndsWith(".B")) plane.B = nib;
        }

        private PlaneVM ResolvePlane(string token)
        {
            // token 形如 "P1.*" ~ "P4.*"
            if (token.Length >= 2 && token[0] == 'P' && char.IsDigit(token[1]))
            {
                int idx = token[1] - '1'; // P1->0
                if (idx >= 0 && idx < Planes.Count) return Planes[idx];
            }
            return null;
        }

        // ---------- 小工具 ----------
        private static byte PackByte(int low, int high)
            => (byte)(((ClampNibble(high) & 0x0F) << 4) | (ClampNibble(low) & 0x0F));

        private static int ClampNibble(int v) => v < 0 ? 0 : (v > 15 ? 15 : v);

        // ---------- Types ----------
        private sealed class PairWrite
        {
            public string Target { get; set; }   // 暫存器名稱
            public string LowSrc { get; set; }   // "P1.R" / 常數
            public string HighSrc { get; set; }  // "P2.G" / 常數
        }

        public sealed class PlaneVM : ViewModelBase
        {
            public int Index { get; }

            public ObservableCollection<int> LevelOptions { get; } =
                new ObservableCollection<int>(Generate(0, 15));

            public int R { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }
            public int G { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }
            public int B { get => GetValue<int>(); set => SetValue(ClampNibble(value)); }

            public PlaneVM(int index) { Index = index; }

            private static int[] Generate(int a, int b)
            {
                int n = b - a + 1;
                var arr = new int[n];
                for (int i = 0; i < n; i++) arr[i] = a + i;
                return arr;
            }
        }
    }
}