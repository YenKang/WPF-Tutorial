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

        // === 綁定屬性 ===
        public string Axis
        {
            get => GetValue<string>();
            set => SetValue(value);
        }

        // ✅ 非三態：true/false；不管值是啥都要能寫入
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

        // === Command ===
        public ICommand ApplyCommand { get; }

        public GrayReverseVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite); // 永遠可寫
            Axis = "H";
        }

        // === 初次/切圖：先吃 JSON；若已 Set 過則嘗試用硬體真值覆蓋 ===
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            if (_cfg == null) return;

            // 1) Axis
            var axisTok = _cfg["axis"];
            var axisStr = axisTok?.ToString()?.Trim()?.ToUpperInvariant();
            Axis = (axisStr == "V") ? "V" : "H";

            // 2) JSON default
            int def = (int?)_cfg["default"] ?? 0;
            Enable = (def != 0);

            Title = (Axis == "V") ? "Vertical Gray Reverse" : "Horizontal Gray Reverse";

            // 3) 如果曾經 Set 過，優先回讀硬體覆蓋顯示（讀不到就保留 JSON 值）
            if (_preferHw)
            {
                TryRefreshFromRegister();
            }
        }

        // === Set：不論 Enable true/false 都會寫入；寫完啟用硬體優先 ===
        private void ExecuteWrite()
        {
            try
            {
                var (reg, mask, shift) = GetAxisSpec(Axis);

                // Rmw8 第 3 參數需先位移好 + mask 對齊
                byte valueShifted = (byte)(((Enable ? 1 : 0) << shift) & mask);
                RegMap.Rmw8(reg, mask, valueShifted);

                // ✅ 之後以硬體真值為主
                _preferHw = true;

                // 寫完立即回讀，讓 UI 與暫存器一致
                TryRefreshFromRegister();

                System.Diagnostics.Debug.WriteLine($"[GrayReverse] Write OK: Axis={Axis}, Value={(Enable ? 1 : 0)}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Write failed: " + ex.Message);
            }
        }

        // === 回讀暫存器（讀不到就不覆蓋 UI） ===
        public void RefreshFromRegister()
        {
            var (reg, mask, shift) = GetAxisSpec(Axis);
            int raw = RegMap.Read8(reg);
            Enable = ((raw & mask) >> shift) != 0;
            System.Diagnostics.Debug.WriteLine($"[GrayReverse] Refresh OK: Axis={Axis}, Enable={Enable}");
        }

        private bool TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); return true; }
            catch (Exception ex)
            {
                // 硬體讀不到 → 保留目前畫面值（JSON 或剛輸入的值），不洗掉
                System.Diagnostics.Debug.WriteLine("[GrayReverse] Refresh skipped: " + ex.Message);
                return false;
            }
        }

        // === 軸向對照：依 datasheet 調整（示例：同一顆暫存器，V=bit5、H=bit4） ===
        private static (string reg, byte mask, int shift) GetAxisSpec(string axis)
        {
            const string REG = "BIST_GrayColor_VH_Reverse";
            return (axis == "V")
                ? (REG, (byte)0x20, 5)  // V → bit5
                : (REG, (byte)0x10, 4); // H → bit4
        }
    }
}