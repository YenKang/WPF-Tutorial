using System;
using System.Globalization;
using System.Linq;
using System.Windows.Input;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayReverseVM : ViewModelBase
    {
        private JObject _cfg;

        // --- 綁定屬性 ---
        public bool Enable
        {
            get => GetValue<bool>();
            set => SetValue(value);
        }

        public string Axis
        {
            get => GetValue<string>();
            set
            {
                SetValue(value);
                Title = (Axis == "V")
                    ? "Vertical Gray Reverse Enable"
                    : "Horizontal Gray Reverse Enable";
            }
        }

        public string Title
        {
            get => GetValue<string>();
            set => SetValue(value);
        }

        public ICommand ApplyCommand { get; }

        public GrayReverseVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
            Axis = "H";  // default
        }

        // === 讀取 JSON 設定 ===
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // Axis
            var axisTok = _cfg["axis"];
            var axis = axisTok != null
                ? axisTok.ToString().Trim().ToUpperInvariant()
                : "H";
            Axis = (axis == "V") ? "V" : "H";

            // Default 值
            int def = (int?)_cfg["default"] ?? 0;
            Enable = (def != 0);

            // 初始化時回讀暫存器，覆蓋實際值
            TryRefreshFromRegister();
        }

        // === 寫入暫存器 ===
        private void ExecuteWrite()
        {
            try
            {
                // 若 JSON 有 writes 直接用 JSON 的規則
                if (_cfg?["writes"] is JArray writes && writes.Count > 0)
                {
                    foreach (JObject w in writes.Cast<JObject>())
                    {
                        string mode = (string)w["mode"] ?? "rmw";
                        string target = (string)w["target"];
                        if (string.IsNullOrEmpty(target)) continue;

                        int mask = ParseInt((string)w["mask"], 0xFF);
                        int shift = (int?)w["shift"] ?? 0;

                        byte cur = RegMap.Read8(target);
                        byte newVal = (byte)((Enable ? 1 : 0) << shift);
                        byte next = (byte)((cur & ~mask) | (newVal & mask));
                        RegMap.Write8(target, next);
                    }
                }
                else
                {
                    // 沒有 writes 就直接寫預設寄存器
                    string regName = (Axis == "V")
                        ? "BIST_GrayColor_VH_Reverse"
                        : "BIST_GrayColor_VH_Reverse";

                    int bit = (Axis == "V") ? 5 : 4; // 依據 datasheet 決定 bit 位置
                    byte cur = RegMap.Read8(regName);
                    byte newVal = (byte)(
                        Enable ? (cur | (1 << bit)) : (cur & ~(1 << bit))
                    );
                    RegMap.Write8(regName, newVal);
                }

                // 寫入後立刻回讀，讓 UI 同步
                TryRefreshFromRegister();
                System.Diagnostics.Debug.WriteLine($"[GrayReverse] Set OK: {Axis}={Enable}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Write failed: " + ex.Message);
            }
        }

        // === 回讀暫存器，更新 CheckBox ===
        public void RefreshFromRegister()
        {
            string regName = (Axis == "V")
                ? "BIST_GrayColor_VH_Reverse"
                : "BIST_GrayColor_VH_Reverse";
            int bit = (Axis == "V") ? 5 : 4;

            int cur = RegMap.Read8(regName);
            Enable = ((cur >> bit) & 0x01) != 0;

            System.Diagnostics.Debug.WriteLine($"[GrayReverse] RefreshFromRegister: {Axis}={Enable}");
        }

        private void TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Skip refresh: " + ex.Message);
            }
        }

        // === 工具函式 ===
        private static int ParseInt(string s, int def = 0)
        {
            if (string.IsNullOrWhiteSpace(s)) return def;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                return int.TryParse(s.Substring(2), NumberStyles.HexNumber, null, out int v) ? v : def;
            return int.TryParse(s, out int v2) ? v2 : def;
        }
    }
}