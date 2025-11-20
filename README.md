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
        private const int TotalMax     = 22;   // 1~22
        private const int TotalDefault = 22;   // 若 JSON 沒給，就用 22

        private string _totalRegister;

        // patternOrder.register 讀進來的暫存器名稱（例如 BIST_PT_ORD0 ~ BIST_PT_ORD21）
        private readonly List<string> _orderTargets  = new List<string>();
        // JSON 給的預設圖片號碼（圖片牆 No.）
        private readonly List<int>    _orderDefaults = new List<int>();

        public IRegisterReadWriteEx RegControl { get; set; }

        public AutoRunVM()
        {
            Debug.WriteLine("[AutoRun] Ctor");

            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // 1~22 給每列 ComboBox 用
            PatternNumberOptions = new ObservableCollection<int>();
            for (int i = 1; i <= 22; i++)
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
        /// Pattern Total：1~22
        /// 不控制 Orders 行數，只表示「實際要寫入幾筆 ORD」。
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
                // 不再動 Orders；Orders 完全由 JSON 建一次就好
            }
        }

        /// <summary>
        /// 固定若干列（通常 22），每列對應一個 ORD 暫存器
        /// </summary>
        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>
        /// 每列 ComboBox 用的數字選項（1~22），代表圖片牆的編號。
        /// </summary>
        public ObservableCollection<int> PatternNumberOptions { get; }

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

            // total 設定
            var t = autoRunControl["total"] as JObject;
            if (t != null)
            {
                _totalRegister = (string)t["register"] ?? string.Empty;
                Debug.WriteLine($"[AutoRun] Total Register Name = {_totalRegister}");

                var def = t["default"] != null ? t["default"].Value<int>() : TotalDefault;
                Debug.WriteLine($"[AutoRun] Total default from JSON = {def}");

                Total = Clamp(def, TotalMin, TotalMax);
                Debug.WriteLine($"[AutoRun] Total after clamp = {Total}");
            }
            else
            {
                Debug.WriteLine("[AutoRun] total section missing in JSON, using defaults.");
                _totalRegister = string.Empty;
                Total          = TotalDefault;
            }

            // patternOrder.register
            _orderTargets.Clear();
            _orderDefaults.Clear();
            Debug.WriteLine("[AutoRun] Parsing patternOrder.register...");

            var order    = autoRunControl["patternOrder"] as JObject;
            var regArray = order != null ? order["register"] as JArray : null;

            if (regArray != null)
            {
                int index = 0;
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;           // e.g. BIST_PT_ORD0
                    var raw     = (string)prop.Value;  // e.g. "1","2",...

                    int defaultNum;
                    if (!int.TryParse(raw, out defaultNum))
                        defaultNum = 1;

                    _orderTargets.Add(regName);
                    _orderDefaults.Add(defaultNum);

                    Debug.WriteLine($"[AutoRun] ORD[{index}] RegName={regName}, DefaultNum={defaultNum}");
                    index++;
                }
            }
            else
            {
                Debug.WriteLine("[AutoRun] patternOrder.register not found or empty.");
            }

            // ✅ 只在這裡建 Orders，一次性建立，不再被 Total 影響
            BuildOrdersFromJson();

            Debug.WriteLine("===== [AutoRun] LoadFrom() end =====");
        }

        /// <summary>
        /// 依照 JSON 的 _orderTargets / _orderDefaults 一次性建立 Orders。
        /// 之後不再變動列數。
        /// </summary>
        private void BuildOrdersFromJson()
        {
            Orders.Clear();

            int count = _orderTargets.Count;
            if (count > 22) count = 22; // 防呆

            Debug.WriteLine($"[AutoRun] BuildOrdersFromJson: count={count}");

            for (int i = 0; i < count; i++)
            {
                string regName = _orderTargets[i];
                int defNum     = (i < _orderDefaults.Count) ? _orderDefaults[i] : 1;
                defNum         = Clamp(defNum, 1, 22);

                var slot = new OrderSlot
                {
                    DisplayNo            = i + 1,
                    RegName              = regName,
                    SelectedPatternNumber = defNum
                };

                Orders.Add(slot);

                Debug.WriteLine(
                    $"[AutoRun] OrderSlot[{i}] → DisplayNo={slot.DisplayNo}, RegName={regName}, DefaultNum={defNum}"
                );
            }
        }

        #endregion

        #region Apply (寫暫存器)

        private void ApplyWrite()
        {
            Debug.WriteLine("===== [AutoRun] ApplyWrite() start =====");

            if (RegControl == null)
            {
                Debug.WriteLine("[AutoRun] ApplyWrite() aborted: RegControl == null");
                return;
            }

            Debug.WriteLine($"[AutoRun] ApplyWrite() with Total={Total}");

            // 1) TOTAL：寫入 (Total-1) 並用 5 bits
            if (!string.IsNullOrEmpty(_totalRegister))
            {
                int t = Clamp(Total, TotalMin, TotalMax) - 1;
                if (t < 0) t = 0;

                uint totalValue = (uint)(t & 0x1F); // 5 bits
                Debug.WriteLine($"[AutoRun] Write TOTAL: Reg={_totalRegister}, Value(dec)={totalValue}, Hex=0x{totalValue:X2}");

                RegControl.SetRegisterByName(_totalRegister, totalValue);
            }
            else
            {
                Debug.WriteLine("[AutoRun] WARNING: _totalRegister is empty, TOTAL not written.");
            }

            // 2) ORD 寫入：Orders 固定 N 列，但只寫前 Total 列
            int maxCount = Math.Min(_orderTargets.Count, Orders.Count);
            int limit    = Math.Min(Total, maxCount);

            Debug.WriteLine($"[AutoRun] ORD: maxCount={maxCount}, limit(Total-based)={limit}");

            for (int i = 0; i < limit; i++)
            {
                var slot = Orders[i];

                string reg = (i < _orderTargets.Count) ? _orderTargets[i] : slot.RegName;
                if (string.IsNullOrEmpty(reg))
                {
                    Debug.WriteLine($"[AutoRun] ORD[{i}] skipped: empty regName.");
                    continue;
                }

                int num        = slot.SelectedPatternNumber;
                int clampedNum = Clamp(num, 1, 22);
                uint ordValue  = (uint)(clampedNum & 0x3F); // 6 bits

                Debug.WriteLine(
                    $"[AutoRun] ORD[{i}] Reg={reg}, SelectedNum={num}, Clamped={clampedNum}, WriteValue(dec)={ordValue}, Hex=0x{ordValue:X2}"
                );

                RegControl.SetRegisterByName(reg, ordValue);
            }

            Debug.WriteLine("===== [AutoRun] ApplyWrite() end =====");
        }

        #endregion

        #region Helpers

        private static int Clamp(int value, int min, int max)
        {
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
        /// 每一列 ComboBox 選到的圖片編號（1~22），直接拿來下暫存器。
        /// </summary>
        public int SelectedPatternNumber
        {
            get { return GetValue<int>(); }
            set
            {
                int old = GetValue<int>();
                if (!SetValue(value)) return;

                Debug.WriteLine($"[AutoRun] OrderSlot DisplayNo={DisplayNo} PatternNumber changed: old={old}, new={value}");
            }
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
}
