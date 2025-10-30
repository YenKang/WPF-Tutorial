using Newtonsoft.Json.Linq;
using System;
using System.Collections.ObjectModel;
using System.Linq;
using System.Windows.Input;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayColorVM : ViewModelBase
    {
        private JObject _cfg;

        // NEW: 寫入成功後，之後切圖/顯示改用硬體真值
        private bool _preferHw = false;

        public ObservableCollection<int> BoolOptions { get; } =
            new ObservableCollection<int>(new[] { 0, 1 });

        public int R { get => GetValue<int>(); set => SetValue(value); }
        public int G { get => GetValue<int>(); set => SetValue(value); }
        public int B { get => GetValue<int>(); set => SetValue(value); }

        public ICommand ApplyCommand { get; }

        public GrayColorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 你原本的 default 載入（保留）
            R = ClampDefault(_cfg["R"] as JObject, 0, 1, 1);  // 依 JSON default
            G = ClampDefault(_cfg["G"] as JObject, 0, 1, 0);
            B = ClampDefault(_cfg["B"] as JObject, 0, 1, 0);

            // NEW: 若曾 Set 過 → 優先用硬體真值覆蓋顯示（讀不到就保留 JSON 值）
            if (_preferHw)
            {
                TryRefreshFromRegister();
            }
        }

        private static int ClampDefault(JObject field, int fbMin, int fbMax, int fbDef)
        {
            if (field == null) return fbDef;
            int min = (int?)field["min"] ?? fbMin;
            int max = (int?)field["max"] ?? fbMax;
            int def = (int?)field["default"] ?? fbDef;
            if (def < min) def = min;
            if (def > max) def = max;
            return def;
        }

        private void ExecuteWrite()
        {
            int r = R & 0x1, g = G & 0x1, b = B & 0x1;

            // 你原本的寫入流程（保留）
            if (!(_cfg?["writes"] is JArray writes) || writes.Count == 0)
            {
                // Fallback：一次 RMW 寫入 0x004F 的 bit[2:0]
                byte cur  = RegMap.Read8("BIST_GrayColor_VH_Reverse");
                byte next = (byte)((cur & ~0x07) | (r << 0) | (g << 1) | (b << 2));
                RegMap.Write8("BIST_GrayColor_VH_Reverse", next);

                // NEW: 寫成功後開始用硬體真值 + 立即回讀同步
                _preferHw = true;
                TryRefreshFromRegister();
                return;
            }

            foreach (JObject w in writes.Cast<JObject>())
            {
                string mode   = (string)w["mode"] ?? "rmw";
                string target = (string)w["target"]; if (string.IsNullOrEmpty(target)) continue;
                string srcStr = (string)w["src"];    // "R"/"G"/"B"/"RGBPacked"/"1"/"0x03"...
                int   srcVal  = ResolveSrc(srcStr, r, g, b);
                if (mode == "write8")
                {
                    RegMap.Write8(target, (byte)(srcVal & 0xFF));
                    continue;
                }

                int mask = ParseInt((string)w["mask"]);      // 支援 "0x.."
                int shift = (int?)w["shift"] ?? 0;

                byte cur  = RegMap.Read8(target);
                byte next = (byte)((cur & ~mask & 0xFF) | (((srcVal << shift) & mask) & 0xFF));
                RegMap.Write8(target, next);
            }

            // NEW: 寫成功後開始用硬體真值 + 立即回讀同步
            _preferHw = true;
            TryRefreshFromRegister();
        }

        // 你原本的回讀（保留）
        public void RefreshFromRegister()
        {
            // 這裡示範直接從 0x004F 回讀 bit[2:0]，若你實際分散在多暫存器，照既有邏輯讀回即可
            int reg = RegMap.Read8("BIST_GrayColor_VH_Reverse");
            R = (reg & 0x01) != 0 ? 1 : 0;
            G = (reg & 0x02) != 0 ? 1 : 0;
            B = (reg & 0x04) != 0 ? 1 : 0;

            System.Diagnostics.Debug.WriteLine(
                $"[GrayColor] Read BIST_GrayColor_VH_Reverse=0x{reg:X2} → R={R}, G={G}, B={B}");
        }

        // NEW: 包一層 try，不覆蓋 UI
        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayColor] RefreshFromRegister skipped: " + ex.Message);
                return false;
            }
        }

        private static int ResolveSrc(string src, int r, int g, int b)
        {
            if (string.IsNullOrEmpty(src)) return 0;
            switch (src)
            {
                case "R": return r;
                case "G": return g;
                case "B": return b;
                case "RGBPacked": return (r & 0x1) | ((g & 0x1) << 1) | ((b & 0x1) << 2);
                default:
                    if (TryParseInt(src, out int v)) return v; // 支援常數
                    return 0;
            }
        }

        private static bool TryParseInt(string s, out int v)
        {
            v = 0;
            if (string.IsNullOrWhiteSpace(s)) return false;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2),
                    System.Globalization.NumberStyles.HexNumber, null, out v);
            return int.TryParse(s, out v);
        }

        private static int ParseInt(string s)
        {
            return TryParseInt(s, out int v) ? v : 0;
        }
    }
}