<!-- 🔹 FCNT1 -->
<StackPanel Orientation="Horizontal" Margin="0,12,0,4">
    <TextBlock Text="FCNT1"
               VerticalAlignment="Center"
               Margin="0,0,8,0" />

    <xctk:IntegerUpDown Width="120"
                        Value="{Binding FCNT1_Value,
                                        Mode=TwoWay,
                                        UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>

<!-- 🔹 FCNT2 -->
<StackPanel Orientation="Horizontal" Margin="0,0,0,8">
    <TextBlock Text="FCNT2"
               VerticalAlignment="Center"
               Margin="0,0,8,0" />

    <xctk:IntegerUpDown Width="120"
                        Value="{Binding FCNT2_Value,
                                        Mode=TwoWay,
                                        UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>

=================

using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Diagnostics;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private const int TotalMin     = 1;
        private const int TotalMax     = 22;
        private const int TotalDefault = 22;

        private string _totalRegister;

        // JSON 給的 patternOrder.register 名稱，例如 BIST_PT_ORD0 ~ BIST_PT_ORD21
        private readonly List<string> _orderTargets = new List<string>();

        // FCNT1 / FCNT2 設定
        private readonly FcntConfig _fcnt1 = new FcntConfig();
        private readonly FcntConfig _fcnt2 = new FcntConfig();

        public IRegisterReadWriteEx RegControl { get; set; }

        public AutoRunVM()
        {
            Debug.WriteLine("[AutoRun] Ctor");

            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // ComboBox 選項：0~21
            PatternNumberOptions = new ObservableCollection<int>();
            for (int i = 0; i <= 21; i++)
            {
                PatternNumberOptions.Add(i);
            }

            Total = TotalDefault;
        }

        #region Public Properties

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// Pattern Total：1~22，只決定要寫幾筆 ORD，不動 Orders 行數。
        /// </summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                var raw = value;
                var v   = Clamp(raw, TotalMin, TotalMax);
                var old = GetValue<int>();

                if (!SetValue(v)) return;

                Debug.WriteLine($"[AutoRun] Total changed: old={old}, ui={raw}, clamped={v}");
            }
        }

        /// <summary>
        /// 每一列 Pattern Order slot（通常 22 列）
        /// </summary>
        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>
        /// ComboBox 選項：0~21
        /// </summary>
        public ObservableCollection<int> PatternNumberOptions { get; }

        /// <summary>
        /// FCNT1 的值（會 clamp 在 JSON min/max 範圍）
        /// </summary>
        public int FCNT1_Value
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, _fcnt1.Min, _fcnt1.Max);
                if (!SetValue(v)) return;
                _fcnt1.Value = v;
                Debug.WriteLine($"[AutoRun] FCNT1_Value set to {v} (min={_fcnt1.Min}, max={_fcnt1.Max})");
            }
        }

        /// <summary>
        /// FCNT2 的值（會 clamp 在 JSON min/max 範圍）
        /// </summary>
        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, _fcnt2.Min, _fcnt2.Max);
                if (!SetValue(v)) return;
                _fcnt2.Value = v;
                Debug.WriteLine($"[AutoRun] FCNT2_Value set to {v} (min={_fcnt2.Min}, max={_fcnt2.Max})");
            }
        }

        public IRelayCommand ApplyCommand { get; }

        #endregion

        #region JSON Load

        public void LoadFrom(JObject autoRunControl)
        {
            Debug.WriteLine("===== [AutoRun] LoadFrom() start =====");

            if (autoRunControl == null)
            {
                Debug.WriteLine("[AutoRun] LoadFrom() aborted: autoRunControl == null");
                return;
            }

            Title = (string)autoRunControl["title"] ?? "Auto Run";
            Debug.WriteLine($"[AutoRun] Title = {Title}");

            // total 區
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;

                int def = t["default"] != null
                    ? t["default"].Value<int>()
                    : TotalDefault;

                Total = Clamp(def, TotalMin, TotalMax);

                Debug.WriteLine(
                    $"[AutoRun] TotalRegister={_totalRegister}, Default={def}, AfterClamp={Total}"
                );
            }
            else
            {
                Debug.WriteLine("[AutoRun] total section missing, use default.");
                _totalRegister = string.Empty;
                Total          = TotalDefault;
            }

            // patternOrder.register → 只拿 register 名稱，不再吃 JSON 的 default 值
            _orderTargets.Clear();
            Debug.WriteLine("[AutoRun] Parsing patternOrder.register...");

            var order    = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;  // e.g. BIST_PT_ORD0
                    _orderTargets.Add(regName);
                }
            }
            else
            {
                Debug.WriteLine("[AutoRun] patternOrder.register not found or empty.");
            }

            // 如果 JSON 沒給任何 register，就用預設 BIST_PT_ORD0~21
            if (_orderTargets.Count == 0)
            {
                Debug.WriteLine("[AutoRun] _orderTargets empty, fallback to BIST_PT_ORD0~21.");
                for (int i = 0; i < 22; i++)
                {
                    _orderTargets.Add($"BIST_PT_ORD{i}");
                }
            }

            // 讀取 fcnt1
            var f1 = autoRunControl["fcnt1"] as JObject;
            if (f1 != null)
            {
                _fcnt1.RegName = (string)f1["register"] ?? string.Empty;
                _fcnt1.Min     = ParseHex((string)f1["min"],     0x01E);
                _fcnt1.Max     = ParseHex((string)f1["max"],     0x258);
                _fcnt1.Value   = ParseHex((string)f1["default"], 0x03C);

                FCNT1_Value = _fcnt1.Value;

                Debug.WriteLine(
                    $"[AutoRun] FCNT1: Reg={_fcnt1.RegName}, Min=0x{_fcnt1.Min:X}, Max=0x{_fcnt1.Max:X}, Default=0x{_fcnt1.Value:X}"
                );
            }
            else
            {
                Debug.WriteLine("[AutoRun] fcnt1 not found in JSON.");
            }

            // 讀取 fcnt2
            var f2 = autoRunControl["fcnt2"] as JObject;
            if (f2 != null)
            {
                _fcnt2.RegName = (string)f2["register"] ?? string.Empty;
                _fcnt2.Min     = ParseHex((string)f2["min"],     0x01E);
                _fcnt2.Max     = ParseHex((string)f2["max"],     0x258);
                _fcnt2.Value   = ParseHex((string)f2["default"], 0x01E);

                FCNT2_Value = _fcnt2.Value;

                Debug.WriteLine(
                    $"[AutoRun] FCNT2: Reg={_fcnt2.RegName}, Min=0x{_fcnt2.Min:X}, Max=0x{_fcnt2.Max:X}, Default=0x{_fcnt2.Value:X}"
                );
            }
            else
            {
                Debug.WriteLine("[AutoRun] fcnt2 not found in JSON.");
            }

            // 一次性建立 Orders，並印出你截圖那種 log
            BuildOrdersFromTargets();

            Debug.WriteLine("===== [AutoRun] LoadFrom() end =====");
        }

        /// <summary>
        /// 根據 _orderTargets 一次性建立 Orders：
        ///  PatternOrder[1] → RegName=BIST_PT_ORD0, DefaultNum=0
        ///  PatternOrder[2] → RegName=BIST_PT_ORD1, DefaultNum=1
        ///  ...
        ///  PatternOrder[22] → RegName=BIST_PT_ORD21, DefaultNum=21
        /// </summary>
        private void BuildOrdersFromTargets()
        {
            Orders.Clear();

            int count = _orderTargets.Count;
            if (count > 22) count = 22;

            for (int i = 0; i < count; i++)
            {
                int displayIndex = i + 1; // PatternOrder[1..22]
                string regName   = _orderTargets[i];
                int defaultNum   = i;     // 0~21

                var slot = new OrderSlot
                {
                    DisplayNo             = displayIndex,
                    RegName               = regName,
                    SelectedPatternNumber = defaultNum
                };

                Orders.Add(slot);

                Debug.WriteLine(
                    $"[AutoRun] PatternOrder[{displayIndex}]  RegName={regName}, DefaultNum={defaultNum}"
                );
            }
        }

        #endregion

        #region Apply Write

        private void ApplyWrite()
        {
            Debug.WriteLine("===== [AutoRun] ApplyWrite() start =====");

            if (RegControl == null)
            {
                Debug.WriteLine("[AutoRun] RegControl == null");
                return;
            }

            Debug.WriteLine($"[AutoRun] ApplyWrite() with Total={Total}");

            // (1) TOTAL：仍用 (Total - 1) 規則，但不做 bit mask
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                int t = Total - 1;
                if (t < 0) t = 0;

                uint totalRaw = (uint)t;

                Debug.WriteLine(
                    $"[AutoRun] Write TOTAL: Reg={_totalRegister}, RawValue(dec)={totalRaw}, Hex=0x{totalRaw:X2}"
                );

                RegControl.SetRegisterByName(_totalRegister, totalRaw);
            }
            else
            {
                Debug.WriteLine("[AutoRun] WARNING: _totalRegister is empty, TOTAL not written.");
            }

            // (2) ORD：完全 raw（不 clamp、不 mask）
            int limit = Math.Min(Total, Orders.Count);
            Debug.WriteLine($"[AutoRun] ORD limit = {limit} (Orders.Count={Orders.Count})");

            for (int i = 0; i < limit; i++)
            {
                var slot = Orders[i];

                int raw = slot.SelectedPatternNumber; // 完全 raw
                uint finalValue = (uint)raw;

                Debug.WriteLine(
                    $"[AutoRun] ORD[{i}] Reg={slot.RegName}, RawNum={raw}, WriteRaw=0x{finalValue:X2}"
                );

                RegControl.SetRegisterByName(slot.RegName, finalValue);
            }

            // (3) FCNT1 / FCNT2：用 JSON min/max clamp 後的值寫入（避免超出 spec）
            if (!string.IsNullOrEmpty(_fcnt1.RegName))
            {
                uint v1 = (uint)_fcnt1.Value;
                Debug.WriteLine(
                    $"[AutoRun] Write FCNT1: Reg={_fcnt1.RegName}, Value(dec)={v1}, Hex=0x{v1:X3}"
                );
                RegControl.SetRegisterByName(_fcnt1.RegName, v1);
            }

            if (!string.IsNullOrEmpty(_fcnt2.RegName))
            {
                uint v2 = (uint)_fcnt2.Value;
                Debug.WriteLine(
                    $"[AutoRun] Write FCNT2: Reg={_fcnt2.RegName}, Value(dec)={v2}, Hex=0x{v2:X3}"
                );
                RegControl.SetRegisterByName(_fcnt2.RegName, v2);
            }

            Debug.WriteLine("===== [AutoRun] ApplyWrite() end =====");
        }

        #endregion

        #region Helpers

        private static int Clamp(int value, int min, int max)
        {
            if (min == 0 && max == 0) return value; // 如果 min/max 還沒初始化，就當作 raw
            if (value < min) return min;
            if (value > max) return max;
            return value;
        }

        private static int ParseHex(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();

            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);

            int val;
            if (int.TryParse(s,
                             System.Globalization.NumberStyles.HexNumber,
                             null, out val))
                return val;

            if (int.TryParse(s, out val))
                return val;

            return fallback;
        }

        #endregion
    }

    public sealed class OrderSlot : ViewModelBase
    {
        public int DisplayNo
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// ComboBox 選到的數字（0~21），直接拿來當暫存器值。
        /// </summary>
        public int SelectedPatternNumber
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// 對應 JSON patternOrder.register 的暫存器名稱（例如 BIST_PT_ORD0）。
        /// </summary>
        public string RegName
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }
    }

    /// <summary>
    /// FCNT 設定封裝
    /// </summary>
    public sealed class FcntConfig
    {
        public string RegName;
        public int Min;
        public int Max;
        public int Value;
    }
}
