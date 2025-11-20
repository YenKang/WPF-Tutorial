## AutoRun.vs

```
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // TOTAL 邊界：你說要 1~21，所以這裡把 Max 改成 21
        private const int TotalMinDefault = 1;
        private const int TotalMaxDefault = 21;
        private const int TotalDefault    = 5;

        // 每個 ORD 只收 6 bits，TOTAL 只收 5 bits（原本就有的）
        private const byte OrdValueMask   = 0x3F;
        private const byte TotalValueMask = 0x1F;

        // JSON: total.register / trigger.target / trigger.value
        private string _totalTarget;
        private string _triggerTarget;
        private byte   _triggerValue;

        // patternOrder.register 解析後的暫存器名稱 & 預設索引
        private readonly List<string> _orderTargets  = new List<string>();
        private readonly List<int>    _orderDefaults = new List<int>();

        // FCNT1 / FCNT2 設定
        private readonly FcntConfig _fcnt1 = new FcntConfig();
        private readonly FcntConfig _fcnt2 = new FcntConfig();

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
            Total = TotalDefault;
        }

        #region Public Properties

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>目前選擇要循環幾張圖（1..21），變更時會同步調整 Orders 筆數</summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                ResizeOrders(value);
            }
        }

        /// <summary>ORD 對應的下拉資料列（每列一個暫存器 + 一個 PatOptions 索引）</summary>
        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        /// <summary>PatOptions：所有可選 pattern（Black / White / Red...）</summary>
        public ObservableCollection<PatternOption> PatternOptions { get; }
            = new ObservableCollection<PatternOption>();

        /// <summary>FCNT1 目前數值（UI 綁這個）</summary>
        public int FCNT1_Value
        {
            get { return GetValue<int>(); }
            set
            {
                // 依 JSON min/max 做 clamp
                var v = Clamp(value, _fcnt1.Min, _fcnt1.Max);
                SetValue(v);
                _fcnt1.Value = v;
            }
        }

        /// <summary>FCNT2 目前數值（UI 綁這個）</summary>
        public int FCNT2_Value
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, _fcnt2.Min, _fcnt2.Max);
                SetValue(v);
                _fcnt2.Value = v;
            }
        }

        public IRelayCommand ApplyCommand { get; }

        #endregion

        #region JSON Load

        /// <summary>由 Pattern JSON 讀取 autoRunControl 區塊（新版 JSON）</summary>
        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            // 1) Title（如果 JSON 沒給，就用 "Auto Run"）
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) total 區塊：{ "min", "max", "default", "register" }
            var totalObj = autoRunControl["total"] as JObject;
            if (totalObj != null)
            {
                _totalTarget = (string)totalObj["register"] ?? _totalTarget;

                var def = totalObj["default"] != null
                    ? totalObj["default"].Value<int>()
                    : TotalDefault;

                var clamped = Clamp(def, TotalMinDefault, TotalMaxDefault);
                Total = clamped; // 這裡會觸發 ResizeOrders(clamped)
            }
            else
            {
                Total = TotalDefault;
            }

            // 3) patternOrder.register
            //    每一個元素長這樣：{ "BIST_PT_ORD0": "0" } → "0" 是 PatOptions.index
            _orderTargets.Clear();
            _orderDefaults.Clear();

            var orderObj  = autoRunControl["patternOrder"] as JObject;
            var regArray  = orderObj != null ? orderObj["register"] as JArray : null;

            if (regArray != null)
            {
                foreach (var jo in regArray.OfType<JObject>())
                {
                    var prop = jo.Properties().FirstOrDefault();
                    if (prop == null) continue;

                    var regName = prop.Name;
                    var raw     = (string)prop.Value;
                    int defaultIndex;
                    if (!int.TryParse(raw, out defaultIndex))
                    {
                        defaultIndex = 0;
                    }

                    _orderTargets.Add(regName);
                    _orderDefaults.Add(defaultIndex);
                }
            }

            // 4) PatOptions：[{ "index": 0, "name": "Black" }, ...]
            PatternOptions.Clear();
            var optArray = autoRunControl["PatOptions"] as JArray;
            if (optArray != null && optArray.Count > 0)
            {
                foreach (var t in optArray.OfType<JObject>())
                {
                    var idx  = t["index"] != null ? t["index"].Value<int>() : 0;
                    var name = (string)t["name"] ?? idx.ToString();

                    PatternOptions.Add(new PatternOption
                    {
                        Index   = idx,
                        Display = name
                    });
                }
            }

            // 5) fcnt1
            var f1 = autoRunControl["fcnt1"] as JObject;
            if (f1 != null)
            {
                _fcnt1.RegName = (string)f1["register"] ?? string.Empty;
                _fcnt1.Min     = ParseHex((string)f1["min"],     0x01E);
                _fcnt1.Max     = ParseHex((string)f1["max"],     0x258);
                _fcnt1.Value   = ParseHex((string)f1["default"], 0x03C);
                FCNT1_Value    = _fcnt1.Value; // 也會同步到 UI
            }

            // 6) fcnt2
            var f2 = autoRunControl["fcnt2"] as JObject;
            if (f2 != null)
            {
                _fcnt2.RegName = (string)f2["register"] ?? string.Empty;
                _fcnt2.Min     = ParseHex((string)f2["min"],     0x01E);
                _fcnt2.Max     = ParseHex((string)f2["max"],     0x258);
                _fcnt2.Value   = ParseHex((string)f2["default"], 0x01E);
                FCNT2_Value    = _fcnt2.Value;
            }

            // 7) trigger（如果有的話就沿用舊邏輯）
            var trig = autoRunControl["trigger"] as JObject;
            if (trig != null)
            {
                _triggerTarget = (string)trig["target"] ?? _triggerTarget;
                _triggerValue  = ParseHexByte((string)trig["value"], 0x80);
            }

            // 8) 在有 _orderTargets / _orderDefaults 之後，再跑一次 ResizeOrders
            //    讓 Orders[i].RegName / SelectedIndex 套入 JSON 預設
            ResizeOrders(Total);
        }

        #endregion

        #region ApplyWrite

        /// <summary>將目前設定寫入暫存器並觸發 Auto-Run</summary>
        public void ApplyWrite()
        {
            // Step 1: 寫入 TOTAL
            if (!string.IsNullOrEmpty(_totalTarget))
            {
                // Reg 定義是 0-based，所以要寫 (Total - 1)，再做 mask
                var encoded = Math.Max(0, Total - 1) & TotalValueMask;
                RegMap.Write8(_totalTarget, (byte)encoded);
            }

            // Step 2: 寫入每一格 ORD（使用 Orders[i].RegName + SelectedIndex）
            var count = Math.Min(Total, Orders.Count);
            for (int i = 0; i < count; i++)
            {
                var slot = Orders[i];
                var reg  = slot.RegName;
                if (string.IsNullOrEmpty(reg)) continue;

                var code = (byte)(slot.SelectedIndex & OrdValueMask);
                RegMap.Write8(reg, code);
            }

            // Step 2.5: 寫入 FCNT1 / FCNT2
            // 新 JSON 只給 "register": "BIST_FCNT1"，這裡假設
            // 實際暫存器仍為 baseName_LO / baseName_HI，
            // 並沿用原本 JSON 的 mask / shift 設定（寫死在 code 裡）
            if (!string.IsNullOrEmpty(_fcnt1.RegName))
            {
                var v1 = Clamp(FCNT1_Value, _fcnt1.Min, _fcnt1.Max);
                WriteFcnt10Bit(_fcnt1.RegName, v1, 0x03, 8);   // 原本 fc1 的 mask / shift
            }

            if (!string.IsNullOrEmpty(_fcnt2.RegName))
            {
                var v2 = Clamp(FCNT2_Value, _fcnt2.Min, _fcnt2.Max);
                WriteFcnt10Bit(_fcnt2.RegName, v2, 0x0F, 2);   // 原本 fc2 的 mask / shift
            }

            // Step 3: 觸發 Auto-Run（如果有 trigger 設定）
            if (!string.IsNullOrEmpty(_triggerTarget))
            {
                RegMap.Write8(_triggerTarget, _triggerValue);
            }
        }

        /// <summary>
        /// 將 10-bit 的 FCNT 值拆成 baseName_LO / baseName_HI 兩個暫存器：
        /// low 直接寫 8 bits，high 依照 mask/shift 寫入。
        /// </summary>
        private static void WriteFcnt10Bit(string baseName, int value10Bit, byte mask, int shift)
        {
            if (string.IsNullOrEmpty(baseName))
                return;

            var lowName  = baseName + "_LO";
            var highName = baseName + "_HI";

            // 低位：直接寫入 8 bits
            RegMap.Write8(lowName, (byte)(value10Bit & 0xFF));

            // 高位：用 RMW 寫入特定位元
            if (mask != 0)
            {
                var hi = (byte)((value10Bit >> shift) & mask);
                RegMap.Rmw8(highName, mask, hi);
            }
        }

        #endregion

        #region Helpers

        private void ResizeOrders(int desiredCount)
        {
            // clamp 在 1..21 之間
            desiredCount = Clamp(desiredCount, TotalMinDefault, TotalMaxDefault);

            // 增加到指定筆數
            while (Orders.Count < desiredCount)
            {
                Orders.Add(new OrderSlot());
            }

            // 減少到指定筆數
            while (Orders.Count > desiredCount)
            {
                Orders.RemoveAt(Orders.Count - 1);
            }

            // 填 DisplayNo / RegName / SelectedIndex（如果 JSON 有給）
            for (int i = 0; i < Orders.Count; i++)
            {
                var slot = Orders[i];
                slot.DisplayNo = i + 1;

                if (i < _orderTargets.Count)
                {
                    slot.RegName = _orderTargets[i];
                }

                if (i < _orderDefaults.Count)
                {
                    slot.SelectedIndex = _orderDefaults[i];
                }
            }
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static byte ParseHexByte(string hexOrNull, byte fallback)
        {
            if (string.IsNullOrWhiteSpace(hexOrNull)) return fallback;

            var s = hexOrNull.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);

            byte result;
            return byte.TryParse(s,
                                 System.Globalization.NumberStyles.HexNumber,
                                 null,
                                 out result)
                ? result
                : fallback;
        }

        private static int ParseHex(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;

            var t = s.Trim();
            if (t.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                t = t.Substring(2);

            int result;
            return int.TryParse(t,
                                System.Globalization.NumberStyles.HexNumber,
                                null,
                                out result)
                ? result
                : fallback;
        }

        #endregion

        #region Inner Types

        /// <summary>PatOptions: index + 顯示名稱（Black / White / Red...）</summary>
        public sealed class PatternOption
        {
            public int Index  { get; set; }
            public string Display { get; set; }
        }

        /// <summary>一列圖片順序下拉所需資料。</summary>
        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            /// <summary>對應的 PatOptions.index（UI SelectedValue 綁這個）</summary>
            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            /// <summary>這一列對應的暫存器名稱，例如 "BIST_PT_ORD0"</summary>
            public string RegName
            {
                get { return GetValue<string>(); }
                set { SetValue(value); }
            }
        }

        /// <summary>FCNT 設定封裝</summary>
        private sealed class FcntConfig
        {
            public string RegName;
            public int Value;
            public int Min;
            public int Max;
        }

        #endregion
    }
}
```

