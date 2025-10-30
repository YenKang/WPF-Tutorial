using System;
using System.Linq;
using System.Windows.Input;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayColorVM : ViewModelBase
    {
        private JObject _cfg;
        private bool _preferHw = false;   // Set 成功後 → 之後顯示一律以硬體真值為主

        // ====== 綁定屬性 ======
        public bool R { get => GetValue<bool>(); set => SetValue(value); } // Red enable
        public bool G { get => GetValue<bool>(); set => SetValue(value); } // Green enable
        public bool B { get => GetValue<bool>(); set => SetValue(value); } // Blue enable

        public string Title { get => GetValue<string>(); set => SetValue(value); }

        public ICommand ApplyCommand { get; }

        public GrayColorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite); // 永遠可執行（不論 R/G/B）
            Title = "Gray Color Channel Enable";
        }

        // ====== 入口：載入 JSON 預設；若曾 Set 過則優先回讀硬體 ======
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 1) JSON defaults（若缺省就當 0）
            if (_cfg["fields"] is JArray fields)
            {
                foreach (var f in fields.OfType<JObject>())
                {
                    string key = (string)f["key"] ?? "";
                    int def = (int?)f["default"] ?? 0;
                    switch (key.ToUpperInvariant())
                    {
                        case "R": R = def != 0; break;
                        case "G": G = def != 0; break;
                        case "B": B = def != 0; break;
                    }
                }
            }

            // 2) 曾經 Set 過 → 以硬體真值覆蓋（讀不到就保留 JSON）
            if (_preferHw)
            {
                TryRefreshFromRegister();
            }
        }

        // ====== Set：永遠寫入 R/G/B 三個 bit；寫完切硬體優先並立即回讀 ======
        private void ExecuteWrite()
        {
            try
            {
                // 寫入 R/G/B（不論 true/false 都會寫）
                TryWriteBit(GetSpec("R"), R);
                TryWriteBit(GetSpec("G"), G);
                TryWriteBit(GetSpec("B"), B);

                // 寫入成功 → 後續用硬體真值
                _preferHw = true;

                // 立刻回讀以同步 UI
                TryRefreshFromRegister();

                System.Diagnostics.Debug.WriteLine($"[GrayColor] Write OK: R={(R?1:0)} G={(G?1:0)} B={(B?1:0)}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayColor] Write failed: " + ex.Message);
            }
        }

        // ====== 回讀硬體：僅在成功時覆蓋 UI，失敗就保持目前值 ======
        public void RefreshFromRegister()
        {
            bool r = R, g = G, b = B; // 暫存目前 UI（避免失敗時被洗掉）

            if (TryReadBit(GetSpec("R"), out bool rv)) r = rv;
            if (TryReadBit(GetSpec("G"), out bool gv)) g = gv;
            if (TryReadBit(GetSpec("B"), out bool bv)) b = bv;

            R = r; G = g; B = b;

            System.Diagnostics.Debug.WriteLine($"[GrayColor] Refresh OK: R={(R?1:0)} G={(G?1:0)} B={(B?1:0)}");
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayColor] Refresh skipped: " + ex.Message);
                return false;
            }
        }

        // ====== JSON 規約優先；若沒有就落回預設映射 ======
        private (string target, byte mask, int shift) GetSpec(string key)
        {
            // 1) 先找 JSON writes：支援 key/name = "R"/"G"/"B"
            if (_cfg?["writes"] is JArray writes)
            {
                foreach (var w in writes.OfType<JObject>())
                {
                    string wk = ((string)w["key"] ?? (string)w["name"] ?? "").Trim().ToUpperInvariant();
                    if (wk == key.ToUpperInvariant())
                    {
                        string target = (string)w["target"];
                        int mask = ParseInt((string)w["mask"], 0);
                        int shift = (int?)w["shift"] ?? InferShiftFromMask(mask);
                        if (!string.IsNullOrEmpty(target) && mask != 0)
                            return (target, (byte)mask, shift);
                    }
                }
            }

            // 2) 預設映射（請依你的實際暫存器調整）
            // 假設三色位於同一顆寄存器：BIST_GrayColor_ChannelEnable
            // R=bit0 (0x01), G=bit1 (0x02), B=bit2 (0x04)
            const string FALLBACK_REG = "BIST_GrayColor_ChannelEnable";
            return key.ToUpperInvariant() switch
            {
                "R" => (FALLBACK_REG, (byte)0x01, 0),
                "G" => (FALLBACK_REG, (byte)0x02, 1),
                "B" => (FALLBACK_REG, (byte)0x04, 2),
                _   => (FALLBACK_REG, (byte)0x00, 0) // 不應到這裡
            };
        }

        // ====== 寫/讀 bit（RMW/READ） ======
        private static bool TryWriteBit((string target, byte mask, int shift) spec, bool value)
        {
            if (string.IsNullOrEmpty(spec.target) || spec.mask == 0) return false;
            byte shifted = (byte)(((value ? 1 : 0) << spec.shift) & spec.mask);
            RegMap.Rmw8(spec.target, spec.mask, shifted);
            return true;
        }

        private static bool TryReadBit((string target, byte mask, int shift) spec, out bool value)
        {
            value = false;
            if (string.IsNullOrEmpty(spec.target) || spec.mask == 0) return false;
            int raw = RegMap.Read8(spec.target);
            value = ((raw & spec.mask) >> spec.shift) != 0;
            return true;
        }

        // ====== 小工具 ======
        private static int ParseInt(string s, int def = 0)
        {
            if (string.IsNullOrWhiteSpace(s)) return def;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out int v) ? v : def;
            return int.TryParse(s, out int v2) ? v2 : def;
        }

        private static int InferShiftFromMask(int mask)
        {
            // 例如 0x04 -> shift=2
            int shift = 0;
            while (((mask >> shift) & 0x01) == 0 && shift < 8) shift++;
            return shift;
        }
    }
}