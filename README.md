using System;
using System.Collections.ObjectModel;
using System.Globalization;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ====== 靜態快取：跨頁/切圖仍保留上次設定 ======
        private static class AutoRunCache
        {
            public static bool HasMemory;
            public static int Total;
            public static int[] Orders = new int[0];
        }

        // ====== 常數（index 位數） ======
        private const byte OrdValueMask   = 0x3F; // 每一格 ORD 只收 6 bits（0..63）
        private const byte TotalValueMask = 0x1F; // TOTAL 只收 5 bits（0..31）→ 代表 1..32

        // ====== JSON 目標位址（由 LoadFrom 讀入）======
        private string _totalTarget;          // e.g. "BIST_PT_TOTAL"
        private string[] _orderTargets = new string[0]; // e.g. ORD0..ORD21

        // ====== View 綁定集合 ======
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ====== 標題 ======
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        // ====== Pattern 總數（改變時同步調整 Orders 個數） ======
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                ResizeOrders(value);
            }
        }

        // ====== 指令 ======
        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        // ─────────────────────────────────────────────────────────
        //  載入 JSON（預設 UI 狀態；若有快取就覆寫 UI）
        // ─────────────────────────────────────────────────────────
        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            // 1) 標題
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) Pattern 選單（index/name）
            PatternOptions.Clear();
            var optArr = autoRunControl["options"] as JArray;
            if (optArr != null)
            {
                foreach (var t in optArr)
                {
                    var o = t as JObject;
                    if (o == null) continue;

                    int idx  = o["index"] != null ? o["index"].Value<int>() : 0;
                    string name = (string)o["name"] ?? idx.ToString(CultureInfo.InvariantCulture);
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // 3) Total 選項（由 JSON 的 min/max 組列）
            var totalObj = autoRunControl["total"] as JObject;
            int min = totalObj != null && totalObj["min"] != null ? totalObj["min"].Value<int>() : 1;
            int max = totalObj != null && totalObj["max"] != null ? totalObj["max"].Value<int>() : 22;
            int def = totalObj != null && totalObj["default"] != null ? totalObj["default"].Value<int>() : 22;
            _totalTarget = totalObj != null ? (string)totalObj["target"] : "BIST_PT_TOTAL";

            if (min < 1) min = 1;
            if (max < min) max = min;

            TotalOptions.Clear();
            for (int i = min; i <= max; i++) TotalOptions.Add(i);

            // 4) Orders 目標暫存器名稱（ORD0..ORDn）
            var ordersObj = autoRunControl["orders"] as JObject;
            _orderTargets = ordersObj != null && ordersObj["targets"] != null
                ? ordersObj["targets"].ToObject<string[]>()
                : new string[0];

            // 5) 先用 JSON default 初始化 UI
            Total = Clamp(def, min, max);
            HydrateOrdersFromJsonDefaults(ordersObj);

            // 6) 若有快取：以快取覆蓋 UI（保留上次設定）
            if (AutoRunCache.HasMemory)
                RestoreFromCache();
        }

        // ─────────────────────────────────────────────────────────
        //  套用（寫入暫存器 + 寫入快取）
        // ─────────────────────────────────────────────────────────
        private void ApplyWrite()
        {
            try
            {
                // Step 1: 寫 TOTAL（硬體採 0 表示 1 張 → n-1）
                if (!string.IsNullOrEmpty(_totalTarget))
                {
                    int encoded = Math.Max(0, Total - 1) & TotalValueMask;
                    RegMap.Write8(_totalTarget, (byte)encoded);
                    System.Diagnostics.Debug.WriteLine(
                        $"[AutoRun] Write TOTAL: UI={Total} -> raw=0x{encoded:X2}");
                }

                // Step 2: 寫各張 ORD
                int count = Math.Min(Total, _orderTargets.Length);
                for (int i = 0; i < count; i++)
                {
                    var target = _orderTargets[i];
                    if (string.IsNullOrEmpty(target)) continue;

                    byte code = (byte)(Orders[i].SelectedIndex & OrdValueMask);
                    RegMap.Write8(target, code);
                    System.Diagnostics.Debug.WriteLine(
                        $"[AutoRun] Write {target}: idx={Orders[i].SelectedIndex} (0x{code:X2})");
                }

                // Step 3: 寫入快取（保留 UI 狀態）
                SaveToCache();
                System.Diagnostics.Debug.WriteLine("[AutoRun] 設定已寫入暫存器並快取完成。");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] ApplyWrite error: " + ex.Message);
            }
        }

        // ─────────────────────────────────────────────────────────
        //  內部：依 JSON 預設填滿 Orders（僅初始化用）
        // ─────────────────────────────────────────────────────────
        private void HydrateOrdersFromJsonDefaults(JObject ordersObj)
        {
            // 清空並建立固定數量的 Order slots
            ResizeOrders(Total);

            // 若 JSON 有 defaults 陣列，依序帶入；否則就按照索引順序 0,1,2...
            int[] defaults = ordersObj != null && ordersObj["defaults"] != null
                ? ordersObj["defaults"].ToObject<int[]>()
                : null;

            for (int i = 0; i < Orders.Count; i++)
            {
                int defIndex = defaults != null && i < defaults.Length ? defaults[i] : i;
                Orders[i].SelectedIndex = defIndex;
            }
        }

        // ─────────────────────────────────────────────────────────
        //  內部：寫入快取／由快取還原
        // ─────────────────────────────────────────────────────────
        private void SaveToCache()
        {
            AutoRunCache.Total = Total;
            AutoRunCache.Orders = new int[Orders.Count];
            for (int i = 0; i < Orders.Count; i++)
                AutoRunCache.Orders[i] = Orders[i].SelectedIndex;
            AutoRunCache.HasMemory = true;
        }

        private void RestoreFromCache()
        {
            // 安全界限：Total 要符合目前 TotalOptions 的範圍
            int min = TotalOptions.Count > 0 ? TotalOptions[0] : 1;
            int max = TotalOptions.Count > 0 ? TotalOptions[TotalOptions.Count - 1] : Math.Max(1, AutoRunCache.Total);
            int total = Clamp(AutoRunCache.Total, min, max);

            Total = total;               // 會自動呼叫 ResizeOrders(total)
            for (int i = 0; i < Orders.Count && i < AutoRunCache.Orders.Length; i++)
                Orders[i].SelectedIndex = AutoRunCache.Orders[i];

            System.Diagnostics.Debug.WriteLine("[AutoRun] 已從快取還原上次設定。");
        }

        // ─────────────────────────────────────────────────────────
        //  工具：調整 OrderSlot 個數（維持 DisplayNo 連號）
        // ─────────────────────────────────────────────────────────
        private void ResizeOrders(int desired)
        {
            if (desired < 1) desired = 1;

            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // ====== 供 ItemsSource 使用的資料型別 ======
        public sealed class PatternOption
        {
            public int Index { get; set; }
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }
        }
    }
}