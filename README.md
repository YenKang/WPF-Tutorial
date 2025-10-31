using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ----- state -----
        private JObject _cfg;
        private bool _preferHw;       // 只要按過 Set 就會變 true
        private bool _isHydrating;    // 初始化／回讀期間抑制副作用
        private int _minTotal = 1;
        private int _maxTotal = 22;

        // ----- UI sources -----
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ----- bindables -----
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
                // 保證不會拋例外，避免紅框
                int clamped = Clamp(value, _minTotal, _maxTotal);
                if (!SetValue(clamped)) return;

                if (_isHydrating) return; // 初始化/回讀期間不動結構
                try
                {
                    ResizeOrders(clamped, hydrateNewFromHw: _preferHw);
                }
                catch (Exception ex)
                {
                    System.Diagnostics.Debug.WriteLine("[AutoRun] Total setter resize error: " + ex.Message);
                }
            }
        }

        // ----- command -----
        public ICommand ApplyCommand { get; }

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        // ---------- lifecycle ----------
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = preferHw;

            _isHydrating = true;
            try
            {
                // 1) Title
                Title = (string)autoRunControl["title"] ?? "Auto Run";

                // 2) Options (index -> name)
                PatternOptions.Clear();
                var optArr = autoRunControl["options"] as JArray;
                if (optArr != null)
                {
                    foreach (var t in optArr)
                    {
                        var o = t as JObject;
                        if (o == null) continue;
                        int idx = o["index"] != null ? o["index"].Value<int>() : 0;
                        string name = (string)o["name"] ?? idx.ToString();
                        PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                    }
                }

                // 3) TotalOptions
                var totalObj = autoRunControl["total"] as JObject;
                _minTotal = totalObj?["min"] != null ? totalObj["min"].Value<int>() : 1;
                _maxTotal = totalObj?["max"] != null ? totalObj["max"].Value<int>() : 22;
                int def = totalObj?["default"] != null ? totalObj["default"].Value<int>() : 22;

                TotalOptions.Clear();
                int lo = Math.Min(_minTotal, _maxTotal);
                int hi = Math.Max(_minTotal, _maxTotal);
                for (int i = lo; i <= hi; i++) TotalOptions.Add(i);

                // 4) 先用 JSON default 建結構（避免 UI 綁定紅框）
                SetValue(nameof(Total), Clamp(def, _minTotal, _maxTotal)); // 不觸發 ResizeOrders
                ResizeOrders(Total, hydrateNewFromHw: false);             // 全部先用 0 填

                // 5) 讀硬體覆蓋（若使用者曾按過 Set 或明確指定 preferHw）
                if (_preferHw)
                {
                    RefreshFromHardware();
                }
                else
                {
                    // 初次：若 JSON 有 orders.defaults，可灌入
                    var ordersObj = autoRunControl["orders"] as JObject;
                    int[] defaults = ordersObj?["defaults"] != null ? ordersObj["defaults"].ToObject<int[]>() : new int[0];
                    for (int i = 0; i < Orders.Count; i++)
                        Orders[i].SelectedIndex = i < defaults.Length ? defaults[i] : (i < PatternOptions.Count ? PatternOptions[i].Index : 0);
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] LoadFrom error: " + ex.Message);
            }
            finally
            {
                _isHydrating = false;
            }
        }

        // ---------- HW read ----------
        public void RefreshFromHardware()
        {
            _isHydrating = true;
            try
            {
                // total: 寫入 n-1；讀回需 +1
                byte raw = RegMap.Read8("BIST_PT_TOTAL");
                int total = Clamp(((int)(raw & 0x1F)) + 1, _minTotal, _maxTotal);

                SetValue(nameof(Total), total);      // 不觸發 resize
                ResizeOrders(total, hydrateNewFromHw: false); // 先確保槽數

                for (int i = 0; i < total; i++)
                {
                    int idx;
                    if (TryReadOrderIndex(i, out idx))
                        Orders[i].SelectedIndex = idx;
                    else
                        Orders[i].SelectedIndex = 0;
                }

                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware OK. total=" + total);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
            finally
            {
                _isHydrating = false;
            }
        }

        private static bool TryReadOrderIndex(int slot, out int idx)
        {
            idx = 0;
            if (slot < 0 || slot > 21) return false; // 只有 ORD0..ORD21
            try
            {
                string reg = "BIST_PT_ORD" + slot;
                byte v = RegMap.Read8(reg);
                idx = v & 0x3F; // 低 6 bits
                return true;
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Read ORD" + slot + " fail: " + ex.Message);
                return false;
            }
        }

        // ---------- HW write ----------
        public void ApplyWrite()
        {
            try
            {
                int total = Clamp(Total, _minTotal, _maxTotal);

                // Step 1: TOTAL = UI-1
                byte encoded = (byte)((total - 1) & 0x1F);
                RegMap.Rmw8("BIST_PT_TOTAL", 0x1F, encoded);
                System.Diagnostics.Debug.WriteLine("[AutoRun] Write TOTAL: UI=" + total + " -> raw=0x" + encoded.ToString("X2"));

                // Step 2: ORD0..(total-1)
                for (int i = 0; i < total; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte code = (byte)(Orders[i].SelectedIndex & 0x3F);
                    RegMap.Write8(reg, code);
                    System.Diagnostics.Debug.WriteLine("[AutoRun] Write " + reg + " = idx=" + code);
                }

                _preferHw = true;
                RefreshFromHardware(); // 寫完立即回讀確認
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] ApplyWrite error: " + ex.Message);
            }
        }

        // ---------- helpers ----------
        private void ResizeOrders(int desired, bool hydrateNewFromHw)
        {
            desired = Clamp(desired, _minTotal, _maxTotal);

            // 擴充
            while (Orders.Count < desired)
            {
                var slotIndex = Orders.Count; // 新增槽的 index
                var slot = new OrderSlot { DisplayNo = slotIndex + 1, SelectedIndex = 0 };

                if (hydrateNewFromHw && TryReadOrderIndex(slotIndex, out int hwIdx))
                    slot.SelectedIndex = hwIdx;

                Orders.Add(slot);
            }
            // 縮減
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            // 重新標號（UI 顯示 1-based）
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // ---------- inner types ----------
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