using System;
using System.Collections.ObjectModel;
using System.Windows;
using System.Windows.Data;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ===== 常數 / 掩碼 =====
        private const int MaxOrders = 22;         // ORD0..ORD21
        private const byte TotalMask = 0x1F;      // BIST_PT_TOTAL[4:0] = (Total - 1)
        private const byte OrdMask   = 0x3F;      // BIST_PT_ORDx 低 6 bit = pattern index

        // ===== 狀態 =====
        private JObject _cfg;
        private bool _preferHw;                   // true：以硬體真值為主
        private bool _isHydrating;                // 批次回填/重繫結期間抑制 re-entrant
        private int _minOrders = 1;               // 由 JSON total.min（夾到 1..22）
        private int _maxOrders = MaxOrders;       // 由 JSON total.max（夾到 1..22）

        // ===== 綁定集合（永不 new）=====
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ===== 綁定屬性 =====
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>Pattern Total（和 ComboBox.SelectedValue 綁定）。</summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                var clamped = Clamp(value, _minOrders, _maxOrders);
                if (SetValue(clamped))
                    ResizeOrders(clamped); // 安全版，內含 UI thread + DeferRefresh
            }
        }

        public ICommand ApplyCommand { get; }

        // ===== ctor =====
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            // 先撐出 ItemsSource，避免初載紅框
            if (TotalOptions.Count == 0)
                for (int v = 1; v <= MaxOrders; v++) TotalOptions.Add(v);
            if (Total == 0) Total = 1;
        }

        // ===== 對外 API =====
        /// <summary>載入 JSON。preferHw=true 時，載完改以硬體真值覆寫 UI。</summary>
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = preferHw;

            // 1) Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) PatternOptions（index/name 對照）
            PatternOptions.Clear();
            var optArr = autoRunControl["options"] as JArray;
            if (optArr != null)
            {
                foreach (var t in optArr)
                {
                    var o = t as JObject; if (o == null) continue;
                    int idx = o["index"] != null ? o["index"].Value<int>() : 0;
                    string name = (string)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // 3) TotalOptions（先建清單 → 再設 Total，避免紅框）
            var totalObj = autoRunControl["total"] as JObject;
            _minOrders = totalObj != null && totalObj["min"] != null ? totalObj["min"].Value<int>() : 1;
            _maxOrders = totalObj != null && totalObj["max"] != null ? totalObj["max"].Value<int>() : MaxOrders;
            int jsonDef = totalObj != null && totalObj["default"] != null ? totalObj["default"].Value<int>() : MaxOrders;

            _minOrders = Clamp(_minOrders, 1, MaxOrders);
            _maxOrders = Clamp(_maxOrders, 1, MaxOrders);
            if (_minOrders > _maxOrders) { _minOrders = 1; _maxOrders = MaxOrders; }

            TotalOptions.Clear();
            for (int i = _minOrders; i <= _maxOrders; i++) TotalOptions.Add(i);

            // 4) 先用 JSON 完整 hydrate（畫面立刻有內容）
            Total = Clamp(jsonDef, _minOrders, _maxOrders);
            HydrateOrdersFromJsonDefaults();

            // 5) 若 preferHw：延後到 UI Loaded 後回讀硬體覆寫（根治時序/紅框）
            var disp = Application.Current?.Dispatcher;
            if (_preferHw && disp != null)
            {
                disp.BeginInvoke(new Action(() =>
                {
                    RefreshFromHardware();
                }), System.Windows.Threading.DispatcherPriority.Loaded);
            }
        }

        /// <summary>寫 TOTAL 與 ORD0.. ，然後轉為 preferHw 並回讀確認。</summary>
        public void Apply()
        {
            try
            {
                int count = Clamp(Total, _minOrders, _maxOrders);

                // TOTAL：0=1 張
                byte totalRaw = (byte)((count - 1) & TotalMask);
                RegMap.Rmw8("BIST_PT_TOTAL", TotalMask, totalRaw);
                System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL UI={count} -> raw=0x{totalRaw:X2}");

                // ORD0..ORD{count-1}
                for (int i = 0; i < count && i < Orders.Count; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte code = (byte)(Orders[i].SelectedIndex & OrdMask);
                    RegMap.Write8(reg, code);
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Write {reg} = 0x{code:X2} ({LookupName(code)})");
                }

                _preferHw = true;
                RefreshFromHardware();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply error: " + ex.Message);
            }
        }

        /// <summary>回讀 BIST_PT_TOTAL 與 BIST_PT_ORDx，批次覆寫 UI。</summary>
        public void RefreshFromHardware()
        {
            try
            {
                // TOTAL
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int totalHw = Clamp(((int)totalRaw & TotalMask) + 1, _minOrders, _maxOrders);

                SafeSetTotal(totalHw); // 內部保證在 UI 執行緒且不紅
                // 先讀完所有 ORD
                int count = Math.Min(totalHw, MaxOrders);
                var idxBuf = new int[count];
                for (int i = 0; i < count; i++)
                {
                    string reg = "BIST_PT_ORD" + i;
                    byte v = RegMap.Read8(reg);
                    idxBuf[i] = v & OrdMask;
                }

                // 批次灌值：一次刷新
                var view = CollectionViewSource.GetDefaultView(Orders);
                using (view != null ? view.DeferRefresh() : null)
                {
                    _isHydrating = true;
                    for (int i = 0; i < count && i < Orders.Count; i++)
                    {
                        int idx = OptionExists(idxBuf[i]) ? idxBuf[i] : DefaultOptionIndex();
                        Orders[i].SelectedIndex = idx;
                    }
                    _isHydrating = false;
                }

                _preferHw = true;
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware done.");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ===== 內部 =====

        /// <summary>安全設定 Total：保證在 UI 執行緒，先有清單再設值。</summary>
        private void SafeSetTotal(int v)
        {
            v = Clamp(v, _minOrders, _maxOrders);

            var disp = Application.Current?.Dispatcher;
            if (disp != null && !disp.CheckAccess())
            {
                disp.Invoke(new Action(() => SafeSetTotal(v)));
                return;
            }

            // 若清單意外為空，先補
            if (TotalOptions.Count == 0)
            {
                for (int i = _minOrders; i <= _maxOrders; i++) TotalOptions.Add(i);
            }

            Total = v; // 會觸發 ResizeOrders（安全版）
        }

        /// <summary>用 JSON defaults（或順序對齊）填滿 Orders。</summary>
        private void HydrateOrdersFromJsonDefaults()
        {
            try
            {
                ResizeOrders(Total);

                var ordersObj = _cfg?["orders"] as JObject;
                var defaults = ordersObj != null ? ordersObj["defaults"] as JArray : null;

                var view = CollectionViewSource.GetDefaultView(Orders);
                using (view != null ? view.DeferRefresh() : null)
                {
                    _isHydrating = true;

                    if (defaults != null && defaults.Count > 0)
                    {
                        for (int i = 0; i < Orders.Count && i < defaults.Count; i++)
                        {
                            int idx = defaults[i].Type == JTokenType.Integer ? defaults[i].Value<int>() : 0;
                            Orders[i].SelectedIndex = OptionExists(idx) ? idx : DefaultOptionIndex();
                        }
                    }
                    else
                    {
                        for (int i = 0; i < Orders.Count; i++)
                            Orders[i].SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
                    }

                    _isHydrating = false;
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] HydrateOrdersFromJsonDefaults error: " + ex.Message);
                // 保底確保有列
                if (Orders.Count == 0)
                    Orders.Add(new OrderSlot { DisplayNo = 1, SelectedIndex = DefaultOptionIndex() });
            }
        }

        /// <summary>批次增減列；UI thread + DeferRefresh + 邊界防護。</summary>
        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, _minOrders, _maxOrders);
            if (desired == Orders.Count) return;

            var disp = Application.Current?.Dispatcher;
            if (disp != null && !disp.CheckAccess())
            {
                disp.Invoke(new Action(() => ResizeOrders(desired)));
                return;
            }

            var view = CollectionViewSource.GetDefaultView(Orders);
            IDisposable defer = null;
            try
            {
                if (view != null) defer = view.DeferRefresh();
                _isHydrating = true;

                if (desired > Orders.Count)
                {
                    int toAdd = desired - Orders.Count;
                    for (int k = 0; k < toAdd; k++)
                    {
                        Orders.Add(new OrderSlot
                        {
                            DisplayNo = Orders.Count + 1,
                            SelectedIndex = DefaultOptionIndex()
                        });
                    }
                }
                else
                {
                    int toRemove = Orders.Count - desired;
                    for (int k = 0; k < toRemove; k++)
                        Orders.RemoveAt(Orders.Count - 1);
                }

                for (int i = 0; i < Orders.Count; i++)
                    Orders[i].DisplayNo = i + 1;
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] ResizeOrders exception: " + ex.Message);
                if (Orders.Count == 0)
                    Orders.Add(new OrderSlot { DisplayNo = 1, SelectedIndex = DefaultOptionIndex() });
            }
            finally
            {
                _isHydrating = false;
                if (defer != null) defer.Dispose();
            }
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private bool OptionExists(int idx)
        {
            for (int i = 0; i < PatternOptions.Count; i++)
                if (PatternOptions[i].Index == idx) return true;
            return false;
        }

        private int DefaultOptionIndex()
        {
            return PatternOptions.Count > 0 ? PatternOptions[0].Index : 0;
        }

        private string LookupName(int idx)
        {
            for (int i = 0; i < PatternOptions.Count; i++)
                if (PatternOptions[i].Index == idx)
                    return PatternOptions[i].Display ?? idx.ToString();
            return idx.ToString();
        }

        // ===== 內部型別 =====
        public sealed class PatternOption
        {
            public int Index { get; set; }   // 0..63
            public string Display { get; set; }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            public int DisplayNo
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }

            /// <summary>每列 ComboBox 的 SelectedValue（= pattern index）。</summary>
            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }
        }
    }
}

