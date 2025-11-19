using Newtonsoft.Json.Linq;
using PageBase.ICStructure.Comm;
using System;
using System.Diagnostics;
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

        // 單一暫存器名稱
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

        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public IRegisterReadWriteEx RegControl { get; set; }

        public ICommand ApplyCommand { get; private set; }

        public BWLoopVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        /// <summary>
        /// 從 bwLoopControl JSON 讀取 fcnt2 參數
        /// </summary>
        public void LoadFrom(JObject bwLoopControlNode)
        {
            if (bwLoopControlNode == null)
                return;

            Debug.WriteLine("[BWLoop LoadFrom] ---");

            var fcnt2 = bwLoopControlNode["fcnt2"] as JObject;
            if (fcnt2 == null)
            {
                Debug.WriteLine("[BWLoop LoadFrom] fcnt2 node is null!");
                return;
            }

            _defaultValue = TryParseInt((string)fcnt2["default"], _defaultValue);
            _min          = TryParseInt((string)fcnt2["min"],      _min);
            _max          = TryParseInt((string)fcnt2["max"],      _max);

            // register 名稱
            var regTok = fcnt2["register"];
            var regName = regTok != null ? regTok.ToString().Trim() : string.Empty;
            if (!string.IsNullOrEmpty(regName))
                _registerName = regName;
            else
                _registerName = "BIST_FCNT2";

            Debug.WriteLine($"[BWLoop LoadFrom] range=0x{_min:X3}~0x{_max:X3}, default=0x{_defaultValue:X3}, register={_registerName}");

            if (BWLoopCache.HasMemory)
            {
                FCNT2_Value = BWLoopCache.FCNT2Value;
                Debug.WriteLine($"[BWLoop LoadFrom] use Cache: 0x{BWLoopCache.FCNT2Value:X3}");
            }
            else
            {
                FCNT2_Value = _defaultValue;
                Debug.WriteLine($"[BWLoop LoadFrom] init Default: 0x{_defaultValue:X3}");
            }
        }

        private void Apply()
        {
            if (RegControl == null)
            {
                Debug.WriteLine("[BWLoop Apply] RegControl is NULL!");
                return;
            }

            int clamped = Clamp(FCNT2_Value, _min, _max);

            Debug.WriteLine($"[BWLoop Apply] FCNT2 input=0x{FCNT2_Value:X3}, clamped=0x{clamped:X3}");

            if (clamped != FCNT2_Value)
                FCNT2_Value = clamped;

            Debug.WriteLine($"[BWLoop Apply] Write register={_registerName}, val=0x{clamped:X3}");

            RegControl.SetRegisterByName(_registerName, (uint)clamped);

            BWLoopCache.SaveBWLoop(clamped);

            Debug.WriteLine($"[BWLoop Apply] Saved Cache val=0x{clamped:X3}");
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

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                if (uint.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber,
                                  null, out uv))
                    return (int)uv;

                return fallback;
            }

            if (uint.TryParse(s, out uv))
                return (int)uv;

            return fallback;
        }
    }
}