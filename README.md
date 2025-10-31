using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    // ================================================================
    // 快取容器：記住上次設定（Total、Order、FCNT）
    // ================================================================
    internal static class AutoRunCache
    {
        public static bool HasMemory { get; private set; }
        public static int Total { get; private set; }
        public static int[] OrderIndices { get; private set; } = new int[0];
        public static int Fcnt1 { get; private set; }
        public static int Fcnt2 { get; private set; }

        public static void Save(int total, int[] orderIdxArray, int fcnt1, int fcnt2)
        {
            Total = total;
            OrderIndices = orderIdxArray ?? new int[0];
            Fcnt1 = Clamp10(fcnt1);
            Fcnt2 = Clamp10(fcnt2);
            HasMemory = true;
        }

        public static void Clear()
        {
            HasMemory = false;
            Total = 0;
            OrderIndices = new int[0];
            Fcnt1 = 0;
            Fcnt2 = 0;
        }

        private static int Clamp10(int v)
        {
            if (v < 0) return 0;
            if (v > 0x3FF) return 0x3FF;
            return v;
        }
    }

    // ================================================================
    // AutoRunVM
    // ================================================================
    public sealed class AutoRunVM : ViewModelBase
    {
        // 暫存器相關
        private string _totalTarget;
        private string[] _orderTargets = new string[0];
        private string _triggerTarget;
        private byte _triggerValue;

        // FCNT1 / FCNT2 JSON Default
        private int _fc1Default, _fc2Default;
        private string _fc1LowTarget, _fc1HighTarget;
        private string _fc2LowTarget, _fc2HighTarget;

        // ================================================================
        // 公開屬性
        // ================================================================
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        // FCNT digits (3x2組)
        public int FCNT1_D2 { get { return GetValue<int>(); } set { SetValue(value); } }
        public int FCNT1_D1 { get { return GetValue<int>(); } set { SetValue(value); } }
        public int FCNT1_D0 { get { return GetValue<int>(); } set { SetValue(value); } }
        public int FCNT2_D2 { get { return GetValue<int>(); } set { SetValue(value); } }
        public int FCNT2_D1 { get { return GetValue<int>(); } set { SetValue(value); } }
        public int FCNT2_D0 { get { return GetValue<int>(); } set { SetValue(value); } }

        public string Title { get { return GetValue<string>(); } set { SetValue(value); } }
        public ICommand ApplyCommand { get; }

        // ================================================================
        // 建構子
        // ================================================================
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        // ================================================================
        // JSON 載入 (含快取回復)
        // ================================================================
        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            // (1) 標題
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // (2) Pattern 選單
            PatternOptions.Clear();
            var optArr = autoRunControl["options"] as JArray;
            if (optArr != null)
            {
                foreach (var t in optArr.OfType<JObject>())
                {
                    int idx = t["index"]?.Value<int>() ?? 0;
                    string name = (string)t["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // (3) Total 設定
            var totalObj = autoRunControl["total"] as JObject;
            int min = totalObj?["min"]?.Value<int>() ?? 1;
            int max = totalObj?["max"]?.Value<int>() ?? 22;
            int def = totalObj?["default"]?.Value<int>() ?? 22;
            _totalTarget = (string)totalObj?["target"] ?? "BIST_PT_TOTAL";

            TotalOptions.Clear();
            for (int i = min; i <= max; i++) TotalOptions.Add(i);

            // (4) Orders 目標
            var ordersObj = autoRunControl["orders"] as JObject;
            _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? new string[0];

            // (5) FCNT1
            if (autoRunControl["fcnt1"] is JObject f1)
            {
                _fc1Default = ParseHex((string)f1["default"], 0x03C);
                if (f1["targets"] is JObject tgt)
                {
                    _fc1LowTarget = (string)tgt["low"];
                    _fc1HighTarget = (string)tgt["high"];
                }
            }

            // (6) FCNT2
            if (autoRunControl["fcnt2"] is JObject f2)
            {
                _fc2Default = ParseHex((string)f2["default"], 0x01E);
                if (f2["targets"] is JObject tgt)
                {
                    _fc2LowTarget = (string)tgt["low"];
                    _fc2HighTarget = (string)tgt["high"];
                }
            }

            // (7) Trigger
            if (autoRunControl["trigger"] is JObject trig)
            {
                _triggerTarget = (string)trig["target"];
                _triggerValue = ParseHexByte((string)trig["value"], 0x3F);
            }

            // ============================================================
            // (8) RestoreFromCache：順序調整後版本 (無紅框)
            // ============================================================
            int totalInit = def;
            if (AutoRunCache.HasMemory)
                totalInit = Math.Min(Math.Max(AutoRunCache.Total, min), max);

            // 先建 UI
            Total = totalInit;
            if (Orders.Count == 0) ResizeOrders(totalInit);

            // 再還原 Orders
            if (AutoRunCache.HasMemory && AutoRunCache.OrderIndices.Length > 0)
            {
                for (int i = 0; i < Math.Min(Orders.Count, AutoRunCache.OrderIndices.Length); i++)
                    Orders[i].SelectedIndex = AutoRunCache.OrderIndices[i];
            }

            // 最後還原 FCNT
            if (AutoRunCache.HasMemory)
            {
                SetFcntDigitsFromValue(AutoRunCache.Fcnt1, true);
                SetFcntDigitsFromValue(AutoRunCache.Fcnt2, false);
            }
            else
            {
                SetFcntDigitsFromValue(_fc1Default, true);
                SetFcntDigitsFromValue(_fc2Default, false);
            }
        }

        // ================================================================
        // 寫暫存器 + 快取
        // ================================================================
        public void ApplyWrite()
        {
            if (string.IsNullOrEmpty(_totalTarget)) return;

            // Step 1: 寫總數
            var encoded = Math.Max(0, Total - 1) & 0x1F;
            RegMap.Write8(_totalTarget, (byte)encoded);

            // Step 2: 寫各個 ORD
            for (int i = 0; i < Math.Min(Total, _orderTargets.Length); i++)
            {
                string target = _orderTargets[i];
                if (string.IsNullOrEmpty(target)) continue;
                var code = (byte)(Orders[i].SelectedIndex & 0x3F);
                RegMap.Write8(target, code);
            }

            // Step 3: 寫 FCNT1 / FCNT2
            int f1Val = GetFcntValueFromDigits(true);
            int f2Val = GetFcntValueFromDigits(false);
            WriteFcnt(_fc1LowTarget, _fc1HighTarget, 0x03, 8, f1Val);
            WriteFcnt(_fc2LowTarget, _fc2HighTarget, 0x0C, 10, f2Val);

            // Step 4: 觸發 AutoRun
            if (!string.IsNullOrEmpty(_triggerTarget))
                RegMap.Write8(_triggerTarget, _triggerValue);

            // Step 5: 寫入快取
            var orderIdx = new int[Total];
            for (int i = 0; i < Total; i++) orderIdx[i] = Orders[i].SelectedIndex;
            AutoRunCache.Save(Total, orderIdx, f1Val, f2Val);
        }

        // ================================================================
        // 工具函式
        // ================================================================
        private void ResizeOrders(int desiredCount)
        {
            while (Orders.Count < desiredCount)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            while (Orders.Count > desiredCount)
                Orders.RemoveAt(Orders.Count - 1);

            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private void SetFcntDigitsFromValue(int value10bit, bool isFcnt1)
        {
            int d2 = (value10bit >> 8) & 0x03;
            int d1 = (value10bit >> 4) & 0x0F;
            int d0 = (value10bit >> 0) & 0x0F;
            if (isFcnt1)
            {
                FCNT1_D2 = d2; FCNT1_D1 = d1; FCNT1_D0 = d0;
            }
            else
            {
                FCNT2_D2 = d2; FCNT2_D1 = d1; FCNT2_D0 = d0;
            }
        }

        private int GetFcntValueFromDigits(bool isFcnt1)
        {
            int d2, d1, d0;
            if (isFcnt1)
            {
                d2 = FCNT1_D2; d1 = FCNT1_D1; d0 = FCNT1_D0;
            }
            else
            {
                d2 = FCNT2_D2; d1 = FCNT2_D1; d0 = FCNT2_D0;
            }
            return ((d2 & 0x03) << 8) | ((d1 & 0x0F) << 4) | (d0 & 0x0F);
        }

        private static int ParseHex(string s, int fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);
            return int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out int v) ? v : fallback;
        }

        private static byte ParseHexByte(string s, byte fallback)
        {
            if (string.IsNullOrWhiteSpace(s)) return fallback;
            s = s.Trim();
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);
            return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out byte v) ? v : fallback;
        }

        private static void WriteFcnt(string lowTarget, string highTarget, byte mask, int shift, int value10bit)
        {
            if (string.IsNullOrEmpty(lowTarget)) return;
            RegMap.Write8(lowTarget, (byte)(value10bit & 0xFF));

            if (!string.IsNullOrEmpty(highTarget))
            {
                byte hi = (byte)((value10bit >> shift) & mask);
                RegMap.Rmw8(highTarget, mask, hi);
            }
        }

        // ================================================================
        // 內部結構
        // ================================================================
        public sealed class PatternOption
        {
            public int Index { get; set; }
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo { get { return GetValue<int>(); } set { SetValue(value); } }
            public int SelectedIndex { get { return GetValue<int>(); } set { SetValue(value); } }
        }
    }
}