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
        private bool _preferHw = false; // ✨ 只要 Set 過，就改用硬體真值
        private string _axisKey;

        // === 屬性 ===
        public string Axis
        {
            get => GetValue<string>();
            set => SetValue(value);
        }

        public bool? Enable
        {
            get => GetValue<bool?>();
            set => SetValue(value);
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
            Axis = "H"; // 預設軸向
        }

        // === LoadFrom(JSON) ===
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 1️⃣ 讀 axis
            var axisTok = _cfg["axis"];
            var axis = axisTok != null ? axisTok.ToString().Trim().ToUpperInvariant() : "H";
            Axis = (axis == "V") ? "V" : "H";

            // 2️⃣ 讀 default
            int def = (int?)_cfg["default"] ?? 0;
            Enable = (def != 0) ? true : (bool?)false;

            Title = (Axis == "V")
                ? "Vertical Gray Reverse"
                : "Horizontal Gray Reverse";

            // 3️⃣ 若已經 Set 過，優先讀暫存器真值覆蓋
            if (_preferHw)
            {
                if (!TryRefreshFromRegister())
                {
                    // 讀不到 → 保留 JSON 值（不覆寫）
                    System.Diagnostics.Debug.WriteLine("[GrayReverse] HW not available, keep JSON default");
                }
            }
        }

        // === Execute Write (Set) ===
        private void ExecuteWrite()
        {
            if (!Enable.HasValue)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Skip write: Enable is null");
                return;
            }

            try
            {
                var (reg, mask, shift) = GetAxisSpec(Axis);
                byte valueShifted = (byte)(((Enable.Value ? 1 : 0) << shift) & mask);
                RegMap.Rmw8(reg, mask, valueShifted);

                // ✅ 寫入成功後，改用硬體真值
                _preferHw = true;

                // 寫完立即回讀同步（確保 UI 與暫存器一致）
                TryRefreshFromRegister();

                System.Diagnostics.Debug.WriteLine($"[GrayReverse] Write success: Axis={Axis}, Enable={Enable}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Write failed: " + ex.Message);
            }
        }

        // === 回讀暫存器 ===
        public void RefreshFromRegister()
        {
            try
            {
                var (reg, mask, shift) = GetAxisSpec(Axis);
                int raw = RegMap.Read8(reg);
                Enable = ((raw & mask) >> shift) != 0;

                System.Diagnostics.Debug.WriteLine($"[GrayReverse] Refresh OK: Axis={Axis}, Enable={Enable}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Refresh failed: " + ex.Message);
                throw; // 交給 Try 包
            }
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch { return false; }
        }

        // === 軸向對照 ===
        private static (string reg, byte mask, int shift) GetAxisSpec(string axis)
        {
            const string REG = "BIST_GrayColor_VH_Reverse";
            return (axis == "V")
                ? (REG, (byte)0x20, 5)  // bit5
                : (REG, (byte)0x10, 4); // bit4
        }
    }
}