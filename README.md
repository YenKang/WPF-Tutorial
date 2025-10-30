using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數（若 JSON 有覆蓋，以 JSON 為準） =====
        private const int TotalMinDefault = 1;
        private const int TotalMaxDefault = 22;
        private const int TotalDefDefault = 22;

        // BIST_PT_TOTAL 的值只有 5 bits
        private const byte TotalValueMask = 0x1F;

        // ===== JSON 解析後的目標暫存器 =====
        private string _totalTarget;               // e.g. "BIST_PT_TOTAL"
        private string[] _orderTargets = Array.Empty<string>();  // e.g. ORD0..ORD21
        private string _triggerTarget;            // 可選：寫完要觸發的暫存器
        private byte _triggerValue;               // 可選：觸發值

        // ===== 由 JSON options 建名單與查表 =====
        public ObservableCollection<PatternOption> PatternOptions { get; } = new();
        private readonly Dictionary<int, string> _indexToName = new();

        // ===== Orders：UI 上 N 個下拉（實際只存 Index） =====
        public ObservableCollection<OrderSlot> Orders { get; } = new();

        // 允許 1..22（或 JSON 指定的 min/max）
        public int Total
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                ResizeOrders(value);
            }
        }

        // 是否優先以硬體真值為準（一旦 Set 過就會設為 true）
        private bool _preferHw;

        public string Title
        {
            get => GetValue<string>();
            private set => SetValue(value);
        }

        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ================== JSON 載入 ==================

        /// <summary>由 Pattern JSON 的 autoRunControl 節點載入。</summary>
        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // ---- total ----
            var totalObj = autoRunControl["total"] as JObject;
            _totalTarget = (string)totalObj?["target"]; // 可為 null（就不寫 total）
            int totalMin = totalObj?["min"]?.Value<int>() ?? TotalMinDefault;
            int totalMax = totalObj?["max"]?.Value<int>() ?? TotalMaxDefault;
            int totalDef = totalObj?["default"]?.Value<int>() ?? TotalDefDefault;

            // 先設總數（會 resize Orders）
            Total = Clamp(totalDef, totalMin, totalMax);

            // ---- orders ----
            var ordersObj = autoRunControl["orders"] as JObject;
            _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? Array.Empty<string>();
            var defaults = ordersObj?["defaults"]?.ToObject<int[]>() ?? Array.Empty<int>();

            Orders.Clear();
            for (int i = 0; i < Total; i++)
            {
                Orders.Add(new OrderSlot
                {
                    DisplayNo = i + 1,
                    SelectedIndex = (i < defaults.Length) ? defaults[i] : 0
                });
            }

            // ---- options → 建名單 & 對照表 ----
            BuildPatternNameMapFromJson(autoRunControl);

            // ---- trigger（可選）----
            var trig = autoRunControl["trigger"] as JObject;
            _triggerTarget = (string)trig?["target"];
            _triggerValue = ParseHexByte((string)trig?["value"], 0x00);

            // 若過去已按過 Set，就優先用硬體真值覆寫 UI
            if (_preferHw) RefreshFromRegister();
        }

        /// <summary>從 JSON 的 options 建 PatternOptions 與 _indexToName。</summary>
        private void BuildPatternNameMapFromJson(JObject autoRunControl)
        {
            PatternOptions.Clear();
            _indexToName.Clear();

            var arr = autoRunControl["options"] as JArray;
            if (arr != null && arr.Count > 0)
            {
                foreach (var t in arr.OfType<JObject>())
                {
                    int idx = t["index"]?.Value<int>() ?? 0;
                    string name = t["name"]?.Value<string>() ?? idx.ToString();
                    _indexToName[idx] = name;
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }
            else
            {
                // 若 JSON 未提供 options，可自行填入 fallback 名單（可留空）
            }
        }

        // ================== Apply（寫入暫存器） ==================

        private void Apply()
        {
            try
            {
                // Step 1: Total（可選）
                if (!string.IsNullOrWhiteSpace(_totalTarget))
                {
                    byte clamped = (byte)(Total & TotalValueMask);
                    RegMap.Write8(_totalTarget, clamped);
                }

                // Step 2: ORD 寫入（依目前 Orders 長度）
                int count = Math.Min(Orders.Count, _orderTargets.Length);
                for (int i = 0; i < count; i++)
                {
                    string target = _orderTargets[i];
                    if (string.IsNullOrWhiteSpace(target)) continue;
                    byte idx = (byte)(Orders[i].SelectedIndex & 0x3F); // 6bits
                    RegMap.Write8(target, idx);
                }

                // Step 3: 觸發（可選）
                if (!string.IsNullOrWhiteSpace(_triggerTarget))
                {
                    RegMap.Write8(_triggerTarget, _triggerValue);
                }

                // 之後以硬體真值為準
                _preferHw = true;

                // 寫完立即回讀同步
                RefreshFromRegister();

                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply OK.");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // ================== Refresh（回讀暫存器 → 覆寫 UI） ==================

        public void RefreshFromRegister()
        {
            try
            {
                if (_orderTargets == null || _orderTargets.Length == 0) return;

                int readCount = Math.Min(Orders.Count, _orderTargets.Length);

                for (int i = 0; i < readCount; i++)
                {
                    string target = _orderTargets[i];
                    if (string.IsNullOrWhiteSpace(target)) continue;

                    int raw = RegMap.Read8(target) & 0x3F; // 只收 6bits
                    Orders[i].SelectedIndex = raw;

                    // 方便看 log：轉名稱
                    string name = _indexToName.TryGetValue(raw, out var disp) ? disp : $"Pattern {raw}";
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] ORD{i} = {raw} ({name})");
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromRegister failed: " + ex.Message);
            }
        }

        // ================== 小工具 ==================

        private void ResizeOrders(int desiredCount)
        {
            if (desiredCount < 0) desiredCount = 0;

            while (Orders.Count < desiredCount)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            while (Orders.Count > desiredCount)
                Orders.RemoveAt(Orders.Count - 1);

            // 重排顯示序號
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private static int Clamp(int v, int min, int max)
            => v < min ? min : (v > max ? max : v);

        private static byte ParseHexByte(string hexOrNull, byte fallback)
        {
            if (string.IsNullOrWhiteSpace(hexOrNull)) return fallback;
            var s = hexOrNull.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
            return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var b) ? b : fallback;
        }

        // ================== 內部型別 ==================

        public sealed class PatternOption
        {
            public int Index { get; set; }
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get => GetValue<int>();
                set => SetValue(value);
            }

            // ComboBox SelectedValue 綁在這個 Index
            public int SelectedIndex
            {
                get => GetValue<int>();
                set => SetValue(value);
            }
        }
    }
}