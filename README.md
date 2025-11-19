using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Diagnostics;
using System.Globalization;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayLevelVM : ViewModelBase
    {
        public static class GrayLevelCache
        {
            private static bool _hasMemory;
            private static int _grayLevel;

            public static bool HasMemory
            {
                get { return _hasMemory; }
            }

            public static int GrayLevel
            {
                get { return _grayLevel; }
            }

            public static void SaveGrayLevel(int value)
            {
                _grayLevel = value;
                _hasMemory = true;
            }
        }

        private JObject _cfg;
        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _default = 0x3FF;
        private string _registerName = "BIST_PT_LEVEL";

        public int GrayLevel_Value
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public IRegisterReadWriteEx RegControl { get; set; }

        public ICommand ApplyCommand { get; private set; }

        public GrayLevelVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        }

        /// <summary>
        /// 由 JSON 的 grayLevelControl 讀取 min / max / default / register
        /// </summary>
        public void LoadFrom(object jsonConfig)
        {
            _cfg = jsonConfig as JObject;
            var root = _cfg;
            if (root == null)
                return;

            // grayLevelControl.grayLevel 區塊
            var grayCtrl = root["grayLevel"] as JObject;
            if (grayCtrl == null)
                return;

            _min = ReadHex(grayCtrl, "min", 0x000);
            _max = ReadHex(grayCtrl, "max", 0x3FF);
            _default = ReadHex(grayCtrl, "default", 0x3FF);

            // 新增：從 JSON 讀取 register 名稱，預設為 BIST_PT_LEVEL
            var regToken = grayCtrl["register"];
            var regName = regToken != null ? (string)regToken : null;
            if (!string.IsNullOrEmpty(regName))
            {
                _registerName = regName;
            }
            else
            {
                _registerName = "BIST_PT_LEVEL";
            }

            if (GrayLevelCache.HasMemory)
            {
                GrayLevel_Value = GrayLevelCache.GrayLevel;
            }
            else
            {
                GrayLevel_Value = _default;
            }

            Debug.WriteLine(
                $"[LoadFrom GrayLevel] range 0x{_min:X3} -> 0x{_max:X3}, default=0x{_default:X3}, register={_registerName}");
        }

        private static int ReadHex(JObject obj, string name, int fallback)
        {
            if (obj == null)
                return fallback;

            var token = obj[name];
            if (token == null)
                return fallback;

            var s = (string)token ?? string.Empty;
            s = s.Trim();
            if (s.Length == 0)
                return fallback;

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                s = s.Substring(2);
            }

            int v;
            if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v))
                return fallback;

            return v;
        }

        private static int Clamp(int value, int min, int max)
        {
            if (value < min) return min;
            if (value > max) return max;
            return value;
        }

        /// <summary>
        /// 寫入 10-bit GrayLevel 到 JSON 指定的 register
        /// 由 RegControl.SetRegisterByName 處理實際暫存器細節
        /// </summary>
        private void ExecuteWrite()
        {
            if (_cfg == null)
            {
                MessageBox.Show("GrayLevel JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("RegControl 尚未注入。");
                return;
            }

            int clamped = Clamp(GrayLevel_Value, _min, _max);
            if (clamped != GrayLevel_Value)
            {
                GrayLevel_Value = clamped; // 讓 UI 值也回到合法範圍
            }

            Debug.WriteLine(
                $"[GrayLevel ExecuteWrite] val=0x{clamped:X3}, register={_registerName}");

            // 新版：不再拆 low8/high2、不再使用 RegMap / writes 陣列
            RegControl.SetRegisterByName(_registerName, (uint)clamped);

            GrayLevelCache.SaveGrayLevel(clamped);
        }
    }
}