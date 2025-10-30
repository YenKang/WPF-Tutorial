using System;
using System.Windows.Input;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayReverseVM : ViewModelBase
    {
        private JObject _cfg;
        private bool _preferHw = false; // ✅ 一旦 Set 成功，之後切圖/顯示都優先讀硬體

        // === 屬性 ===
        public string Axis
        {
            get => GetValue<string>();
            set => SetValue(value);
        }

        public bool Enable
        {
            get => GetValue<bool>();
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
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite); // 永遠可執行
            Axis = "H";
        }

        // === 載入 JSON 設定 ===
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 1️⃣ 讀取軸向設定
            var axisTok = _cfg["axis"];
            var axisStr = axisTok?.ToString()?.Trim()?.ToUpperInvariant();
            Axis = (axisStr == "V") ? "V" : "H";

            // 2️⃣ 初始值從 JSON default
            int def = (int?)_cfg["default"] ?? 0;
            Enable = (def != 0);

            Title = (Axis == "V")
                ? "Vertical Gray Reverse"
                : "Horizontal Gray Reverse";

            // 3️⃣ 若曾經 Set 過 → 優先回讀暫存器真值覆蓋顯示
            if (_preferHw)
            {
                TryRefreshFromRegister();
            }
        }

        // === Set 按鈕動作 ===
        private void ExecuteWrite()
        {
            try
            {
                var (reg, mask, shift) = GetAxisSpec(Axis);

                // 寫入 bit 值（不論 true/false 都能寫入）
                byte valueShifted = (byte)(((Enable ? 1 : 0) << shift) & mask);
                RegMap.Rmw8(reg, mask, valueShifted);

                // ✅ 寫完改成硬體優先模式
                _preferHw = true;

                // 寫完立即回讀同步
                TryRefreshFromRegister();

                System.Diagnostics.Debug.WriteLine($"[GrayReverse] Write OK: Axis={Axis}, Value={(Enable ? 1 : 0)}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Write failed: " + ex.Message);
            }
        }

        // === 回讀暫存器 ===
        public void RefreshFromRegister()
        {
            var (reg, mask, shift) = GetAxisSpec(Axis);
            int raw = RegMap.Read8(reg);
            Enable = ((raw & mask) >> shift) != 0;

            System.Diagnostics.Debug.WriteLine($"[GrayReverse] Refresh OK: Axis={Axis}, Enable={Enable}");
        }

        private bool TryRefreshFromRegister()
        {
            try
            {
                RefreshFromRegister();
                return true;
            }
            catch (Exception ex)
            {
                // 硬體讀不到 → 保留 UI 現值（JSON 或使用者上次設定）
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Refresh skipped: " + ex.Message);
                return false;
            }
        }

        // === 軸向對照 ===
        private static (string reg, byte mask, int shift) GetAxisSpec(string axis)
        {
            const string REG = "BIST_GrayColor_VH_Reverse";
            return (axis == "V")
                ? (REG, (byte)0x20, 5)  // V → bit5
                : (REG, (byte)0x10, 4); // H → bit4
        }
    }
}