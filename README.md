using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class BWLoopVM : ViewModelBase
    {
        // JSON 參數
        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _defaultValue = 0x01E;

        // 單一暫存器名稱（由 JSON 的 "register" 決定）
        private string _registerName = "BIST_FCNT2";

        /// <summary>
        /// 儲存使用者上次設定的 FCNT2 值
        /// </summary>
        public static class BWLoopCache
        {
            public static bool HasMemory { get; private set; }

            public static int FCNT2Value { get; private set; }

            public static void SaveBWLoop(int fcnt2Value)
            {
                FCNT2Value = fcnt2Value;
                HasMemory = true;
            }

            public static void Clear()
            {
                HasMemory = false;
                FCNT2Value = 0;
            }
        }

        /// <summary>
        /// UI 綁定用的 FCNT2 數值（0x000 ~ 0x3FF）
        /// </summary>
        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 實際寫暫存器用的介面，由外部注入
        /// </summary>
        public IRegisterReadWriteEx RegControl { get; set; }

        public ICommand ApplyCommand { get; private set; }

        public BWLoopVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        /// <summary>
        /// 從 bwLoopControl JSON 讀取 fcnt2 的 min/max/default/register
        /// </summary>
        public void LoadFrom(JObject bwLoopControlNode)
        {
            if (bwLoopControlNode == null)
                return;

            var fcnt2 = bwLoopControlNode["fcnt2"] as JObject;
            if (fcnt2 == null)
                return;

            _defaultValue = TryParseInt((string)fcnt2["default"], _defaultValue);
            _min          = TryParseInt((string)fcnt2["min"],      _min);
            _max          = TryParseInt((string)fcnt2["max"],      _max);

            // 新 JSON：單一 register
            var regTok = fcnt2["register"];
            var regName = regTok != null ? regTok.ToString().Trim() : string.Empty;
            if (!string.IsNullOrEmpty(regName))
            {
                _registerName = regName;
            }
            else
            {
                _registerName = "BIST_FCNT2"; // 保底名稱
            }

            if (BWLoopCache.HasMemory)
            {
                FCNT2_Value = BWLoopCache.FCNT2Value;
            }
            else
            {
                FCNT2_Value = _defaultValue;
            }
        }

        private void Apply()
        {
            if (RegControl == null)
                return;

            // 1. clamp 在合法範圍
            int value = Clamp(FCNT2_Value, _min, _max);
            if (value != FCNT2_Value)
                FCNT2_Value = value; // 修正 UI 顯示

            // 2. 直接把 10-bit 整數寫進 JSON 指定的暫存器
            RegControl.SetRegisterByName(_registerName, (uint)value);

            // 3. 存入快取，切換圖再回來時沿用
            BWLoopCache.SaveBWLoop(value);
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static int TryParseInt(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s))
                return fallback;

            s = s.Trim();
            uint uv;

            // 支援 "0x..." hex
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                if (uint.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber,
                                  null, out uv))
                {
                    return (int)uv;
                }
                return fallback;
            }

            // 一般十進位
            if (uint.TryParse(s, out uv))
                return (int)uv;

            return fallback;
        }
    }
}