using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private JObject _cfg;
        private bool _preferHw;

        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                if (SetValue(value))
                    ResizeOrders(value);
            }
        }

        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null)
                return;

            _cfg = autoRunControl;
            _preferHw = preferHw;

            // ===== 1️⃣ 標題 =====
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // ===== 2️⃣ Pattern 選項 =====
            PatternOptions.Clear();
            var optArr = autoRunControl["options"] as JArray;
            if (optArr != null)
            {
                foreach (var token in optArr)
                {
                    var o = token as JObject;
                    if (o == null) continue;

                    int idx = o["index"] != null ? o["index"].Value<int>() : 0;
                    string name = (string)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // ===== 3️⃣ Total ComboBox =====
            TotalOptions.Clear();
            var totalObj = autoRunControl["total"] as JObject;
            int min = totalObj?["min"] != null ? totalObj["min"].Value<int>() : 1;
            int max = totalObj?["max"] != null ? totalObj["max"].Value<int>() : 22;
            int def = totalObj?["default"] != null ? totalObj["default"].Value<int>() : 22;
            for (int i = min; i <= max; i++) TotalOptions.Add(i);

            Total = Clamp(def, min, max);

            // ===== 4️⃣ 初始 Orders =====
            ResizeOrders(Total);
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].SelectedIndex = i < PatternOptions.Count ? PatternOptions[i].Index : 0;

            // ===== 5️⃣ 若 preferHw, 則直接讀取硬體 =====
            if (_preferHw)
                RefreshFromHardware();
        }

        public void RefreshFromHardware()
        {
            try
            {
                // --- 讀取 Total ---
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int total = (totalRaw & 0x1F) + 1;
                Total = total;

                // --- 讀取每張圖 ---
                for (int i = 0; i < total; i++)
                {
                    string reg = string.Format("BIST_PT_ORD{0}", i);
                    byte val = RegMap.Read8(reg);
                    int idx = val & 0x3F;
                    EnsureOrderSlot(i).SelectedIndex = idx;
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware done.");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        public void Apply()
        {
            if (Total <= 0) return;

            // 寫 TOTAL (0=1張 → 寫入 n-1)
            byte totalRaw = (byte)((Total - 1) & 0x1F);
            RegMap.Rmw8("BIST_PT_TOTAL", 0x1F, totalRaw);

            // 寫每張圖順序
            for (int i = 0; i < Total; i++)
            {
                string reg = string.Format("BIST_PT_ORD{0}", i);
                byte val = (byte)(Orders[i].SelectedIndex & 0x3F);
                RegMap.Write8(reg, val);
                System.Diagnostics.Debug.WriteLine(
                    string.Format("[AutoRun] Write ORD{0:D2} = idx={1}", i, val));
            }

            _preferHw = true;
            RefreshFromHardware();
        }

        private void ResizeOrders(int desired)
        {
            while (Orders.Count < desired)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1 });
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);
        }

        private OrderSlot EnsureOrderSlot(int i)
        {
            while (Orders.Count <= i)
                Orders.Add(new OrderSlot { DisplayNo = i + 1 });
            return Orders[i];
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // ===== inner class =====

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