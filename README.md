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
        private const int TotalDefault = 22;

        private string _totalRegister;
        private readonly List<string> _orderTargets = new List<string>();

        private readonly FcntConfig _fcnt1 = new FcntConfig();
        private readonly FcntConfig _fcnt2 = new FcntConfig();

        public IRegisterReadWriteEx RegControl { get; set; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // ComboBox 選項 0~21
            PatternNumberOptions = new ObservableCollection<int>();
            for (int i = 0; i <= 21; i++)
                PatternNumberOptions.Add(i);

            Total = TotalDefault;
        }

        #region Properties

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>
        /// Pattern Total：由 XAML UpDown 控制 1~22，不另外 clamp。
        /// 只影響「要寫幾筆 ORD」。
        /// </summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                Debug.WriteLine($"[AutoRun] Total = {value}");
            }
        }

        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>
        /// 每列 ComboBox 的選項 0~21。
        /// </summary>
        public ObservableCollection<int> PatternNumberOptions { get; }

        public int FCNT1_Value
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                _fcnt1.Value = value;
                Debug.WriteLine($"[AutoRun] FCNT1_Value = {value}");
            }
        }

        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                _fcnt2.Value = value;
                Debug.WriteLine($"[AutoRun] FCNT2_Value = {value}");
            }
        }

        public IRelayCommand ApplyCommand { get; }

        #endregion

        #region JSON Load

        public void LoadFrom(JObject autoRunControl)
        {
            Debug.WriteLine("===== [AutoRun] LoadFrom start =====");

            if (autoRunControl == null)
            {
                Debug.WriteLine("[AutoRun] autoRunControl == null");
                return;
            }

            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // total
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;
                var def = t["default"] != null ? t["default"].Value<int>() : TotalDefault;
                Total = def;
                Debug.WriteLine($"[AutoRun] TotalReg={_totalRegister}, Default={def}");
            }

            // patternOrder.register → 只吃 register 名稱
            _orderTargets.Clear();

            var order    = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;
                    _orderTargets.Add(prop.Name);   // e.g. BIST_PT_ORD0
                }
            }

            // fallback：沒給就用 BIST_PT_ORD0~21
            if (_orderTargets.Count == 0)
            {
                for (int i = 0; i < 22; i++)
                    _orderTargets.Add($"BIST_PT_ORD{i}");
            }

            // fcnt1
            var f1 = autoRunControl["fcnt1"] as JObject;
            if (f1 != null)
            {
                _fcnt1.RegName = (string)f1["register"] ?? string.Empty;
                _fcnt1.Min     = ParseHex((string)f1["min"],     0x01E);
                _fcnt1.Max     = ParseHex((string)f1["max"],     0x258);
                _fcnt1.Value   = ParseHex((string)f1["default"], 0x03C);
                FCNT1_Value    = _fcnt1.Value;
            }

            // fcnt2
            var f2 = autoRunControl["fcnt2"] as JObject;
            if (f2 != null)
            {
                _fcnt2.RegName = (string)f2["register"] ?? string.Empty;
                _fcnt2.Min     = ParseHex((string)f2["min"],     0x01E);
                _fcnt2.Max     = ParseHex((string)f2["max"],     0x258);
                _fcnt2.Value   = ParseHex((string)f2["default"], 0x01E);
                FCNT2_Value    = _fcnt2.Value;
            }

            BuildOrdersOnce();

            Debug.WriteLine("===== [AutoRun] LoadFrom end =====");
        }

        private void BuildOrdersOnce()
        {
            Orders.Clear();

            var count = _orderTargets.Count;
            if (count > 22) count = 22;

            for (int i = 0; i < count; i++)
            {
                var displayNo = i + 1;
                var regName   = _orderTargets[i];
                var defaultNum = i; // 0~21

                Orders.Add(new OrderSlot
                {
                    DisplayNo             = displayNo,
                    RegName               = regName,
                    SelectedPatternNumber = defaultNum
                });

                Debug.WriteLine(
                    $"[AutoRun] PatternOrder[{displayNo}] RegName={regName}, DefaultNum={defaultNum}");
            }
        }

        #endregion

        #region Apply

        private void ApplyWrite()
        {
            Debug.WriteLine("===== [AutoRun] ApplyWrite start =====");

            if (RegControl == null)
            {
                Debug.WriteLine("[AutoRun] RegControl == null");
                return;
            }

            // 1) TOTAL：仍採用 (Total - 1)，不再 mask
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                var t = Total - 1;
                if (t < 0) t = 0;

                var totalRaw = (uint)t;
                Debug.WriteLine(
                    $"[AutoRun] TOTAL Reg={_totalRegister}, Raw={totalRaw} (0x{totalRaw:X2})");

                RegControl.SetRegisterByName(_totalRegister, totalRaw);
            }

            // 2) ORD：完全 raw（不 clamp、不 mask）
            var limit = Math.Min(Total, Orders.Count);

            for (int i = 0; i < limit; i++)
            {
                var slot      = Orders[i];
                var raw       = slot.SelectedPatternNumber;
                var finalValue = (uint)raw;

                Debug.WriteLine(
                    $"[AutoRun] ORD[{i}] Reg={slot.RegName}, RawNum={raw}, Write=0x{finalValue:X2}");

                RegControl.SetRegisterByName(slot.RegName, finalValue);
            }

            // 3) FCNT1 / FCNT2：使用目前 FCNT*_Value raw 寫入
            if (!string.IsNullOrEmpty(_fcnt1.RegName))
            {
                var v1 = (uint)FCNT1_Value;
                Debug.WriteLine(
                    $"[AutoRun] FCNT1 Reg={_fcnt1.RegName}, Value={FCNT1_Value} (0x{v1:X3})");
                RegControl.SetRegisterByName(_fcnt1.RegName, v1);
            }

            if (!string.IsNullOrEmpty(_fcnt2.RegName))
            {
                var v2 = (uint)FCNT2_Value;
                Debug.WriteLine(
                    $"[AutoRun] FCNT2 Reg={_fcnt2.RegName}, Value={FCNT2_Value} (0x{v2:X3})");
                RegControl.SetRegisterByName(_fcnt2.RegName, v2);
            }

            Debug.WriteLine("===== [AutoRun] ApplyWrite end =====");
        }

        #endregion

        #region Helpers

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
        /// ComboBox 選到的數字（0~21），直接 raw 寫暫存器。
        /// </summary>
        public int SelectedPatternNumber
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public string RegName
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }
    }

    public sealed class FcntConfig
    {
        public string RegName;
        public int Min;
        public int Max;
        public int Value;
    }
}
