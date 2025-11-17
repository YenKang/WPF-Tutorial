using System;
using System.Diagnostics;
using System.Globalization;
using Newtonsoft.Json.Linq;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class GrayLevelVM : ViewModelBase
    {
        private JObject _cfg;

        private int _min = 0x000;
        private int _max = 0x3FF;
        private int _default = 0x3FF;
        private int _value = 0x3FF;

        public int GrayLevel_Value
        {
            get { return _value; }
            set
            {
                if (_value == value) return;
                _value = value;
                OnPropertyChanged();
            }
        }

        public string GrayLevel_Text
        {
            get { return GetValue<string>(); }
            set
            {
                SetValue(value);

                if (string.IsNullOrWhiteSpace(value))
                    return;

                try
                {
                    var s = value.Trim();

                    // 允許 "0x3FF" 或 "3FF"
                    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                        s = s.Substring(2);

                    int v;
                    if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v))
                        return;

                    // clamp
                    if (v < _min) v = _min;
                    if (v > _max) v = _max;

                    GrayLevel_Value = v;

                    // 標準化為 3 位 HEX
                    var normalized = v.ToString("X3");
                    SetValue(normalized);
                }
                catch
                {
                    // 打字中不彈 error
                }
            }
        }

        public ICommand ApplyCommand { get; private set; }

        public GrayLevelVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
            GrayLevel_Text = "000";
        }

        /// <summary>
        /// 從 JSON 的 grayLevelControl 讀取 min / max / default
        /// </summary>
        public void LoadFrom(object jsonConfig)
        {
            _cfg = jsonConfig as JObject;
            if (_cfg == null)
                return;

            var glCtrl = _cfg["grayLevelControl"]?["grayLevel"] as JObject;
            if (glCtrl == null)
                return;

            _min = ReadHex(glCtrl, "min", 0x000);
            _max = ReadHex(glCtrl, "max", 0x3FF);
            _default = ReadHex(glCtrl, "default", 0x3FF);

            GrayLevel_Value = _default;
            GrayLevel_Text = _default.ToString("X3");

            Debug.WriteLine($"[LoadFrom] GrayLevel: min=0x{_min:X3}, max=0x{_max:X3}, default=0x{_default:X3}");
        }

        private static int ReadHex(JObject obj, string name, int fallback)
        {
            if (obj == null) return fallback;

            var token = obj[name];
            if (token == null) return fallback;

            var s = ((string)token)?.Trim() ?? "";

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);

            int v;
            if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v))
                return fallback;

            return v;
        }

        /// <summary>
        /// 真正寫入暫存器
        /// GrayLevel_Value = 10-bit 數值
        /// low 8 bits → BIST_PT_LEVEL
        /// high 2 bits → BIST_CK_GY_FK_PT bit1~bit0
        /// </summary>
        private void ExecuteWrite()
        {
            if (_cfg == null)
                return;

            byte low8 = (byte)(GrayLevel_Value & 0xFF);          // bit7~0
            byte high2 = (byte)((GrayLevel_Value >> 8) & 0x03);  // bit9~8

            Debug.WriteLine($"[Gray ExecuteWrite] val=0x{GrayLevel_Value:X3}, low=0x{low8:X2}, high2=0x{high2:X1}");

            var writes = _cfg["writes"] as JArray;
            if (writes == null || writes.Count == 0)
            {
                MessageBox.Show("GrayLevel JSON 格式錯誤");
                return;
            }

            for (int i = 0; i < writes.Count; i++)
            {
                JObject w = (JObject)writes[i];
                string mode = (string)w["mode"];
                string target = (string)w["target"];
                if (string.IsNullOrEmpty(target)) continue;

                Debug.WriteLine($"→ rule: mode={mode}, target={target}");

                if (mode == "write8")
                {
                    Debug.WriteLine($"  write8 {target} = 0x{low8:X2}");
                    RegMap.Write8(target, low8);
                }
                else if (mode == "rmw")
                {
                    int mask = Convert.ToInt32((string)w["mask"], 16);  // 0x03
                    int shift = (int?)w["shift"] ?? 0;                   // 0

                    byte cur = RegMap.Read8(target);

                    byte newBits = (byte)((high2 << shift) & mask);

                    byte next = (byte)((cur & ~mask) | newBits);

                    Debug.WriteLine($"  rmw {target}: cur=0x{cur:X2}, next=0x{next:X2}");

                    RegMap.Write8(target, next);
                }
            }
        }
    }
}