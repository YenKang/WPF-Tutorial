using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ---------- fields ----------
        private JObject _cfg;
        private bool _preferHw;          // 是否偏好用硬體真值覆寫 UI
        private int _minOrders = 1;      // 由 JSON total.min
        private int _maxOrders = 22;     // 由 JSON total.max（上限 22）
        private const int MaxHwOrdRegs = 22;  // BIST_PT_ORD0..21 共 22 顆
        private const byte OrdValueMask  = 0x3F; // 每格 ORD 僅 6 bits
        private const byte TotalValueMask= 0x1F; // TOTAL 僅收 5 bits（0=1張）

        // ---------- collections ----------
        public ObservableCollection<int> TotalOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<PatternOption> PatternOptions { get; } = new ObservableCollection<PatternOption>();
        public ObservableCollection<OrderSlot> Orders { get; } = new ObservableCollection<OrderSlot>();

        // ---------- properties ----------
        public string Title
        {
            get { return GetValue<string>(); }
            set { SetValue(value); }
        }

        /// <summary>UI 的「Pattern Total」；SelectedItem 綁到這個 int。</summary>
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                // 先夾在 JSON 範圍
                var clamped = Clamp(value, _minOrders, _maxOrders);
                if (SetValue(clamped))
                {
                    ResizeOrders(clamped);
                }
            }
        }

        public ICommand ApplyCommand { get; }

        // ---------- ctor ----------
        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // ---------- public API ----------
        /// <summary>
        /// 由 JSON 載入 Auto-Run 設定；若 preferHw=true，載入完成後會回讀硬體覆寫 UI。
        /// </summary>
        public void LoadFrom(JObject autoRunControl, bool preferHw = false)
        {
            if (autoRunControl == null) return;

            _cfg = autoRunControl;
            _preferHw = preferHw;

            // 1) Title
            Title = (string)autoRunControl["title"] ?? "Auto Run";

            // 2) PatternOptions（index + name 對照）
            PatternOptions.Clear();
            var optArr = autoRunControl["options"] as JArray;
            if (optArr != null)
            {
                foreach (var t in optArr)
                {
                    var o = t as JObject;
                    if (o == null) continue;

                    int idx  = o["index"]  != null ? o["index"].Value<int>() : 0;
                    string name = (string)o["name"] ?? idx.ToString();
                    PatternOptions.Add(new PatternOption { Index = idx, Display = name });
                }
            }

            // 3) TotalOptions（先建好，避免 UI 綁定找不到候選值而紅框）
            var totalObj = autoRunControl["total"] as JObject;
            _minOrders = totalObj?["min"]     != null ? totalObj["min"].Value<int>() : 1;
            _maxOrders = totalObj?["max"]     != null ? totalObj["max"].Value<int>() : 22;
            int def    = totalObj?["default"] != null ? totalObj["default"].Value<int>() : 22;

            // 邊界保險
            if (_minOrders < 1) _minOrders = 1;
            if (_maxOrders > MaxHwOrdRegs) _maxOrders = MaxHwOrdRegs;
            if (_minOrders > _maxOrders) { _minOrders = 1; _maxOrders = MaxHwOrdRegs; }

            TotalOptions.Clear();
            for (int i = _minOrders; i <= _maxOrders; i++) TotalOptions.Add(i);

            // 4) 先用 JSON 預設把 UI 綁定完整（這步才設 Total，避免紅框）
            Total = Clamp(def, _minOrders, _maxOrders);

            // 5) Orders 初始值（若 JSON 有 orders.defaults 就用，否則用 index 對齊）
            HydrateOrdersFromJsonDefaults();

            // 6) 若指定 preferHw => 回讀硬體覆寫 UI
            if (_preferHw)
            {
                RefreshFromHardware();
            }
        }

        /// <summary>把目前 UI 設定寫進暫存器；寫完即切換為 preferHw 模式並回讀確認。</summary>
        public void Apply()
        {
            try
            {
                if (Total < _minOrders) return;

                // Step 1: 寫 TOTAL（硬體 0 = 1 張，所以寫入 (Total-1)）
                byte encoded = (byte)((Total - 1) & TotalValueMask);
                RegMap.Rmw8("BIST_PT_TOTAL", TotalValueMask, encoded);
                System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL: UI={Total} -> raw=0x{encoded:X2}");

                // Step 2: 寫每張 ORD（最多 22 顆）
                int count = Math.Min(Total, MaxHwOrdRegs);
                for (int i = 0; i < count; i++)
                {
                    string reg = $"BIST_PT_ORD{i}";
                    byte code  = (byte)(Orders[i].SelectedIndex & OrdValueMask);
                    RegMap.Write8(reg, code);
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Write ORD{i:D2}: idx={code} ({LookupName(code)})");
                }

                // 切換為 preferHw，並回讀確認
                _preferHw = true;
                RefreshFromHardware();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply error: " + ex.Message);
            }
        }

        /// <summary>從硬體回讀 TOTAL 與各個 ORD，覆寫 UI。</summary>
        public void RefreshFromHardware()
        {
            try
            {
                // --- TOTAL ---
                byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
                int total = ((int)totalRaw & TotalValueMask) + 1;     // 0 -> 1 張
                total = Clamp(total, _minOrders, _maxOrders);

                // 先把 Total 設好，讓 UI 有正確的行數
                if (Total != total) Total = total; else ResizeOrders(total);

                System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL: raw=0x{totalRaw:X2} -> UI={total}");

                // --- 各張圖 ORD0..ORD(total-1) ---
                int count = Math.Min(total, MaxHwOrdRegs);
                for (int i = 0; i < count; i++)
                {
                    string reg = $"BIST_PT_ORD{i}";
                    byte idxRaw = RegMap.Read8(reg);
                    int idx = idxRaw & OrdValueMask;
                    EnsureOrderSlot(i).SelectedIndex = idx;
                    System.Diagnostics.Debug.WriteLine($"[AutoRun] Read ORD{i:D2}: 0x{idxRaw:X2} -> idx={idx} ({LookupName(idx)})");
                }

                _preferHw = true; // 之後切圖維持用硬體真值
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
            }
        }

        // ---------- helpers ----------
        private void HydrateOrdersFromJsonDefaults()
        {
            // 若 JSON 提供 orders.defaults（index 陣列）就用；否則用遞增 index 當預設
            try
            {
                var ordersObj = _cfg?["orders"] as JObject;
                var defaults  = ordersObj?["defaults"] as JArray;

                ResizeOrders(Total);

                if (defaults != null && defaults.Count > 0)
                {
                    for (int i = 0; i < Orders.Count && i < defaults.Count; i++)
                    {
                        int idx = defaults[i].Type == JTokenType.Integer
                                  ? defaults[i].Value<int>()
                                  : 0;
                        Orders[i].SelectedIndex = idx;
                    }
                }
                else
                {
                    // 無 defaults：用 pattern index 對齊（0,1,2,...）
                    for (int i = 0; i < Orders.Count; i++)
                    {
                        Orders[i].SelectedIndex = (i < PatternOptions.Count)
                                                  ? PatternOptions[i].Index
                                                  : 0;
                    }
                }
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] HydrateOrdersFromJsonDefaults failed: " + ex.Message);
                // 兜底：仍確保 Orders 長度正確
                ResizeOrders(Total);
            }
        }

        /// <summary>調整 Orders 列數；若 _preferHw=true，新增列會嘗試從硬體讀取對應 ORD 值。</summary>
        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, _minOrders, _maxOrders);

            // 增加
            while (Orders.Count < desired)
            {
                int i = Orders.Count;
                var slot = new OrderSlot { DisplayNo = i + 1, SelectedIndex = 0 };

                if (_preferHw && i < MaxHwOrdRegs)
                {
                    // 新增列時，若現在偏好硬體 → 直接讀對應 ORD i
                    try
                    {
                        string reg = $"BIST_PT_ORD{i}";
                        byte idxRaw = RegMap.Read8(reg);
                        int idx = idxRaw & OrdValueMask;
                        slot.SelectedIndex = idx;
                        System.Diagnostics.Debug.WriteLine($"[AutoRun] Resize+ ORD{i:D2} from HW: idx={idx} ({LookupName(idx)})");
                    }
                    catch { /* 忽略硬體讀取失敗，保持 0 */ }
                }
                else
                {
                    // 非 preferHw → 用 JSON 或遞增預設（在 HydrateOrdersFromJsonDefaults 會再覆寫）
                    slot.SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
                }

                Orders.Add(slot);
            }

            // 減少
            while (Orders.Count > desired)
                Orders.RemoveAt(Orders.Count - 1);

            // 重新標號
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].DisplayNo = i + 1;
        }

        private OrderSlot EnsureOrderSlot(int i)
        {
            while (Orders.Count <= i)
                Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });
            return Orders[i];
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private string LookupName(int idx)
        {
            for (int i = 0; i < PatternOptions.Count; i++)
                if (PatternOptions[i].Index == idx)
                    return PatternOptions[i].Display;
            return idx.ToString();
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

            /// <summary>綁定到每列 ComboBox 的 SelectedValue（= pattern index）。</summary>
            public int SelectedIndex
            {
                get { return GetValue<int>(); }
                set { SetValue(value); }
            }
        }
    }
}