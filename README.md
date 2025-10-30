using System;
using System.Collections.ObjectModel;
using System.Linq;
using Newtonsoft.Json.Linq;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // --- 常數與欄位 ---
        private const int TotalMin = 1;
        private const int TotalMax = 22;

        // 由 JSON 帶入（可為空）
        private string _totalTarget = "BIST_PT_TOTAL";

        // 是否優先用硬體真值（按下 Set 後打開）
        private bool _preferHw = false;

        // --- 對外屬性／可供 XAML 綁定 ---
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        // UI: 1..22；寫入時會轉成 0..21
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(Clamp(value, TotalMin, TotalMax))) return;
                ResizeOrders(Total);
            }
        }

        // Pattern 選單（來源：JSON autoRunControl.options）
        public ObservableCollection<PatternOption> PatternOptions { get; } =
            new ObservableCollection<PatternOption>();

        // 可選的 Pattern 數量 1..22
        public ObservableCollection<int> TotalOptions { get; } =
            new ObservableCollection<int>(Enumerable.Range(TotalMin, TotalMax));

        // 真的要寫入/顯示的序列：Order 1..Total
        public ObservableCollection<OrderSlot> Orders { get; } =
            new ObservableCollection<OrderSlot>();

        public ICommand ApplyCommand { get; private set; }

        // --- 建構 ---
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
            Title = "Auto Run";
            Total = 22; // 初始給值；LoadFrom 會覆蓋
            ResizeOrders(Total);
        }

        // --- 對應你的 JSON 結構 ---
        // {
        //   "autoRunControl": {
        //     "title": "Auto Run",
        //     "total": { "min":1, "max":22, "default":22, "target":"BIST_PT_TOTAL" },
        //     "options": [ { "index":0, "name":"0.Black" }, ... ],
        //     "orders": { "defaults":[0,1,2,...] } // 可省略
        //   }
        // }
        public void LoadFrom(JObject autoRunControl)
        {
            if (autoRunControl == null) return;

            _preferHw = false; // 初次進頁以 JSON 為準

            // Title
            var t = (string)autoRunControl["title"];
            if (!string.IsNullOrWhiteSpace(t)) Title = t;

            // total 區
            var totalObj = autoRunControl["total"] as JObject;
            if (totalObj != null)
            {
                _totalTarget = (string)totalObj["target"] ?? "BIST_PT_TOTAL";

                int def = (int?)totalObj["default"] ?? 22;
                int min = (int?)totalObj["min"] ?? TotalMin;
                int max = (int?)totalObj["max"] ?? TotalMax;

                // 重新建 TotalOptions（保守處理）
                TotalOptions.Clear();
                var minClamped = Math.Max(TotalMin, min);
                var maxClamped = Math.Min(TotalMax, max);
                for (int i = minClamped; i <= maxClamped; i++)
                    TotalOptions.Add(i);

                Total = Clamp(def, minClamped, maxClamped);
            }
            else
            {
                // fallback
                TotalOptions.Clear();
                for (int i = TotalMin; i <= TotalMax; i++) TotalOptions.Add(i);
                Total = 22;
            }

            // options -> PatternOptions
            PatternOptions.Clear();
            var options = autoRunControl["options"] as JArray;
            if (options != null)
            {
                foreach (var jo in options.OfType<JObject>())
                {
                    var idx = (int?)jo["index"] ?? 0;
                    var name = (string)jo["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }
            else
            {
                // 沒有給 options 的極端情況，至少補 0..63
                for (int i = 0; i < 64; i++)
                    PatternOptions.Add(new PatternOption { Index = i, Display = i.ToString() });
            }

            // orders.defaults -> 初始化 Orders
            var ordersObj = autoRunControl["orders"] as JObject;
            var defaultsArr = ordersObj != null ? ordersObj["defaults"] as JArray : null;

            ResizeOrders(Total);
            if (defaultsArr != null && defaultsArr.Count > 0)
            {
                for (int i = 0; i < Total && i < defaultsArr.Count; i++)
                {
                    int idx = (int?)defaultsArr[i] ?? 0;
                    Orders[i].SelectedIndex = idx;
                }
            }
            else
            {
                // 若未提供 defaults，給前 N 個遞增
                for (int i = 0; i < Total; i++)
                    Orders[i].SelectedIndex = i < PatternOptions.Count ? PatternOptions[i].Index : 0;
            }
        }

        // --- Apply：寫入 BIST ---
        private void Apply()
        {
            // 1) 寫入 TOTAL（UI 1..22 -> HW 0..21）
            int totalClamped = Clamp(Total, TotalMin, TotalMax);
            byte totalHw = (byte)((totalClamped - 1) & 0x1F);
            RegMap.Write8(_totalTarget, totalHw);

            // 2) 寫入每格 ORD（0..Total-1）
            for (int i = 0; i < totalClamped; i++)
            {
                int patIndex = Orders[i].SelectedIndex;           // 0..63
                byte val = (byte)(patIndex & 0x3F);               // 6 bits
                RegMap.Write8(OrdName(i), val);
            }

            // 3) 清尾端避免殘值
            for (int i = totalClamped; i < 22; i++)
                RegMap.Write8(OrdName(i), 0x00);

            // 4) 之後都優先用硬體值回填
            _preferHw = true;
            RefreshFromHardware();

            System.Diagnostics.Debug.WriteLine(
                $"[AutoRun] Write OK: Total(UI)={Total} -> HW={totalHw:X2}");
        }

        // --- 切圖/重進頁：若 _preferHw=true 就讀硬體回填 ---
        public void RefreshFromHardware()
        {
            if (!_preferHw) return;

            try
            {
                int hwTotalRaw = RegMap.Read8(_totalTarget) & 0x1F; // 0..21
                int uiTotal = hwTotalRaw + 1;                       // 1..22
                uiTotal = Clamp(uiTotal, TotalMin, TotalMax);

                Total = uiTotal;            // 會觸發 ResizeOrders
                ResizeOrders(Total);

                for (int i = 0; i < Total; i++)
                {
                    int ordVal = RegMap.Read8(OrdName(i)) & 0x3F;   // 6 bits
                    Orders[i].SelectedIndex = ordVal;               // 綁 SelectedValuePath="Index"
                }

                System.Diagnostics.Debug.WriteLine(
                    $"[AutoRun] Refresh HW -> Total={Total}, ORD0={Orders[0].SelectedIndex}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware failed: " + ex.Message);
            }
        }

        // --- 工具 ---
        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static string OrdName(int i)
        {
            // 你的 JSON register 區有定義 BIST_PT_ORD0 ~ BIST_PT_ORD21
            return "BIST_PT_ORD" + i;
        }

        private void ResizeOrders(int desired)
        {
            if (desired < TotalMin) desired = TotalMin;
            if (desired > TotalMax) desired = TotalMax;

            // 增
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

            // 減
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            // 更新顯示序號（1-based）
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        // --- 內部型別 ---
        public sealed class PatternOption
        {
            public int Index { get; set; }     // 0..63
            public string Display { get; set; } // UI 顯示字串
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            // 綁定 ComboBox：ItemsSource=PatternOptions, SelectedValuePath="Index"
            // 所以這裡放實際要寫進 ORD 的 Index（0..63）
            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }
        }
    }
}