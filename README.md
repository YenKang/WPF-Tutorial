using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Diagnostics;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayColorVM : ViewModelBase
    {
        private JObject _cfg;

        // R/G/B 範圍與預設
        private int _rMin = 0;
        private int _rMax = 1;
        private int _rDefault = 1;

        private int _gMin = 0;
        private int _gMax = 1;
        private int _gDefault = 0;

        private int _bMin = 0;
        private int _bMax = 1;
        private int _bDefault = 0;

        // 各自對應的暫存器名稱（由 JSON 的 register 欄位決定）
        private string _rRegister = "BIST_GRAY_R_EN";
        private string _gRegister = "BIST_GRAY_G_EN";
        private string _bRegister = "BIST_GRAY_B_EN";

        public ObservableCollection<int> BoolOptions { get; } =
            new ObservableCollection<int>(new[] { 0, 1 });

        public int R
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int G
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int B
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public IRegisterReadWriteEx RegControl { get; set; }

        public ICommand ApplyCommand { get; private set; }

        public GrayColorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        /// <summary>
        /// 從 grayColorControl JSON 讀取 fields.R/G/B 的 min/max/default/register
        /// </summary>
        public void LoadFrom(object jsonCfg)
        {
            _cfg = jsonCfg as JObject;
            var fields = _cfg?["fields"] as JObject;
            if (fields == null)
                return;

            // ---------- R ----------
            var rField = fields["R"] as JObject;
            if (rField != null)
            {
                _rMin = ReadInt(rField, "min", 0);
                _rMax = ReadInt(rField, "max", 1);
                _rDefault = ReadInt(rField, "default", 1);

                var regTok = rField["register"];
                var regName = regTok != null ? (string)regTok : null;
                if (!string.IsNullOrEmpty(regName))
                    _rRegister = regName;
            }

            // ---------- G ----------
            var gField = fields["G"] as JObject;
            if (gField != null)
            {
                _gMin = ReadInt(gField, "min", 0);
                _gMax = ReadInt(gField, "max", 1);
                _gDefault = ReadInt(gField, "default", 0);

                var regTok = gField["register"];
                var regName = regTok != null ? (string)regTok : null;
                if (!string.IsNullOrEmpty(regName))
                    _gRegister = regName;
            }

            // ---------- B ----------
            var bField = fields["B"] as JObject;
            if (bField != null)
            {
                _bMin = ReadInt(bField, "min", 0);
                _bMax = ReadInt(bField, "max", 1);
                _bDefault = ReadInt(bField, "default", 0);

                var regTok = bField["register"];
                var regName = regTok != null ? (string)regTok : null;
                if (!string.IsNullOrEmpty(regName))
                    _bRegister = regName;
            }

            // 初始化目前值（依 JSON default）
            R = Clamp(_rDefault, _rMin, _rMax);
            G = Clamp(_gDefault, _gMin, _gMax);
            B = Clamp(_bDefault, _bMin, _bMax);

            Debug.WriteLine(
                $"[GrayColor LoadFrom] R: min={_rMin}, max={_rMax}, def={_rDefault}, reg={_rRegister}");
            Debug.WriteLine(
                $"[GrayColor LoadFrom] G: min={_gMin}, max={_gMax}, def={_gDefault}, reg={_gRegister}");
            Debug.WriteLine(
                $"[GrayColor LoadFrom] B: min={_bMin}, max={_bMax}, def={_bDefault}, reg={_bRegister}");
        }

        private static int ReadInt(JObject field, string name, int fallback)
        {
            if (field == null)
                return fallback;

            var token = field[name];
            if (token == null)
                return fallback;

            try
            {
                return token.Value<int>();
            }
            catch
            {
                return fallback;
            }
        }

        private static int Clamp(int value, int min, int max)
        {
            if (value < min) return min;
            if (value > max) return max;
            return value;
        }

        /// <summary>
        /// 將 R/G/B 寫入各自的 enable 暫存器（0 或 1）
        /// </summary>
        private void ExecuteWrite()
        {
            if (_cfg == null)
            {
                MessageBox.Show("GrayColor JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("RegControl 尚未注入。");
                return;
            }

            // 先 clamp，順便把 UI 的值也修正進合法範圍
            var rVal = Clamp(R, _rMin, _rMax);
            var gVal = Clamp(G, _gMin, _gMax);
            var bVal = Clamp(B, _bMin, _bMax);

            if (rVal != R) R = rVal;
            if (gVal != G) G = gVal;
            if (bVal != B) B = bVal;

            Debug.WriteLine(
                $"[GrayColor ExecuteWrite] R={rVal}, G={gVal}, B={bVal}, " +
                $"regR={_rRegister}, regG={_gRegister}, regB={_bRegister}");

            // 新版：不再使用 RegMap、mask、shift、writes[]
            RegControl.SetRegisterByName(_rRegister, (uint)rVal);
            RegControl.SetRegisterByName(_gRegister, (uint)gVal);
            RegControl.SetRegisterByName(_bRegister, (uint)bVal);
        }
    }
}