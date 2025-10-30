using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        // ====== 常數 & 掩碼 ======
        private const int TotalMin = 1;
        private const int TotalMax = 22;      // 0..21 共 22 張
        private const byte TotalMask = 0x1F;  // TOTAL 只有 5 bits

        // ====== Reg 名稱（以 JSON registers 里的 key 為準）======
        private const string REG_TOTAL = "BIST_PT_TOTAL";
        private static string OrdName(int i) { return "BIST_PT_ORD" + i; }   // BIST_PT_ORD0..21

        // ====== 內部狀態 ======
        private JObject _cfg;                 // 指向 autoRunControl 節點
        private bool _preferHw = false;       // 套用後、下次切圖優先讀硬體
        private readonly Dictionary<int, string> _nameByIndex = new Dictionary<int, string>();  // 0..21 -> 顯示名

        // ====== UI：Pattern 總數、22 個 order 下拉選單 ======
        public int Total
        {
            get { return GetValue<int>(); }
            set
            {
                var v = Clamp(value, TotalMin, TotalMax);
                if (!SetValue(v)) return;
                ResizeOrders(v);
            }
        }

        public ObservableCollection<OrderSlot> Orders { get; private set; }
        public ObservableCollection<PatternOption> PatternOptions { get; private set; }

        public ICommand ApplyCommand { get; private set; }

        public AutoRunVM()
        {
            Orders = new ObservableCollection<OrderSlot>();
            PatternOptions = new ObservableCollection<PatternOption>();
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // =====================================================================
        // 1) 由 JSON 載入（第一次進來，或你切圖時呼叫）
        // =====================================================================
        public void LoadFrom(JObject autoRunControl)
        {
            _cfg = autoRunControl;

            // 1. 先建立 0..21 的預設名稱
            BuildDefaultNameTable();

            // 2. 合併 JSON options（若有）
            LoadOptionsFromJson(_cfg);

            // 3. 初始 TOTAL
            var total = 22; // default fallback
            var totalNode = _cfg != null ? _cfg["total"] as JObject : null;
            if (totalNode != null)
            {
                var defTok = totalNode["default"];
                if (defTok != null) total = ToInt(defTok, 22);
                var minTok = totalNode["min"];
                var maxTok = totalNode["max"];
                var minV = minTok != null ? ToInt(minTok, TotalMin) : TotalMin;
                var maxV = maxTok != null ? ToInt(maxTok, TotalMax) : TotalMax;
                total = Clamp(total, minV, maxV);
            }
            Total = total;    // 內部會呼叫 ResizeOrders

            // 4. 若 _preferHw=true，優先讀硬體；否則使用 JSON defaults（若 orders.defaults 存在）
            if (_preferHw)
            {
                TryRefreshFromHardware();
            }
            else
            {
                LoadOrderDefaultsFromJson(_cfg);  // 沒有就保持 0,1,2...
            }
        }

        // =====================================================================
        // 2) Apply：把 UI 的 Total 與各 slot 設定寫入暫存器
        // =====================================================================
        private void Apply()
        {
            try
            {
                // Step 1: 寫 TOTAL
                var totalToWrite = (byte)(Total & TotalMask);
                RegMap.Write8(REG_TOTAL, totalToWrite);

                // Step 2: 寫 ORD0..ORD(total-1)
                int count = Orders.Count;
                for (int i = 0; i < count; i++)
                {
                    // SelectedIndex 對應 0..21 的 pattern index
                    int patIdx = Clamp(Orders[i].SelectedIndex, 0, 21);
                    RegMap.Write8(OrdName(i), (byte)patIdx);
                }

                // 打開偏好：下次切圖優先讀硬體
                _preferHw = true;

                // 寫完立即回讀一次，確保 UI 與硬體同步
                TryRefreshFromHardware();

                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply OK: total=" + Total);
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
            }
        }

        // =====================================================================
        // 3) 讀硬體值覆蓋 UI（_preferHw 時呼叫；或 Apply 後呼叫）
        // =====================================================================
        private void RefreshFromHardware()
        {
            // TOTAL
            byte hwTotal = RegMap.Read8(REG_TOTAL);
            int t = hwTotal & TotalMask;
            if (t < TotalMin) t = TotalMin;
            if (t > TotalMax) t = TotalMax;

            // 重新配置 orders 數量
            ResizeOrders(t);

            // ORD slots
            for (int i = 0; i < t; i++)
            {
                byte v = RegMap.Read8(OrdName(i));
                int idx = v & 0x3F;              // 6 bits（datasheet 寫 index[5:0]，但有效為 0..21）
                idx = Clamp(idx, 0, 21);
                Orders[i].SelectedIndex = idx;   // 綁定下拉
            }
        }

        private void TryRefreshFromHardware()
        {
            try { RefreshFromHardware(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[AutoRun] Skip HW refresh: " + ex.Message);
            }
        }

        // =====================================================================
        // 4) JSON 輔助：options 與 defaults
        // =====================================================================
        private void BuildDefaultNameTable()
        {
            _nameByIndex.Clear();
            // 預設名：會被 JSON 覆蓋
            // 你也可以把這裡改成更友好的文字
            for (int i = 0; i <= 21; i++)
            {
                _nameByIndex[i] = i + ".Pattern";
            }
            // 常見預設（可選；若 JSON 有，會覆蓋）
            SetNameIfEmpty(0,  "0.Black");
            SetNameIfEmpty(1,  "1.White");
            SetNameIfEmpty(2,  "2.Red");
            SetNameIfEmpty(3,  "3.Green");
            SetNameIfEmpty(4,  "4.Blue");
            SetNameIfEmpty(5,  "5.Horizontal 16 Gray Scale");
            SetNameIfEmpty(6,  "6.Vertical 16 Gray Scale");
            SetNameIfEmpty(7,  "7.Horizontal 256 Gray Scale");
            SetNameIfEmpty(8,  "8.Vertical 256 Gray Scale");
            SetNameIfEmpty(9,  "9.8 Color Bar");
            SetNameIfEmpty(10, "10.Chess Board with M*N Block");
            SetNameIfEmpty(11, "11.Cursor");
            SetNameIfEmpty(12, "12.One-Pixel On/Off");
            SetNameIfEmpty(13, "13.Two-Pixel On/Off");
            SetNameIfEmpty(14, "14.One Sub-pixel On/Off");
            SetNameIfEmpty(15, "15.Two Sub-pixel On/Off");
            SetNameIfEmpty(16, "16.Crosstalk Center Black");
            SetNameIfEmpty(17, "17.Crosstalk Center White");
            SetNameIfEmpty(18, "18.Flicker Pattern");
            SetNameIfEmpty(19, "19.Eight Color Bar + Gray");
            SetNameIfEmpty(20, "20.Three Color Bar");
            SetNameIfEmpty(21, "21.4 Horizontal Bar (B to W)");
        }

        private void SetNameIfEmpty(int idx, string name)
        {
            string exist;
            if (!_nameByIndex.TryGetValue(idx, out exist) || string.IsNullOrWhiteSpace(exist))
            {
                _nameByIndex[idx] = name;
            }
        }

        private void LoadOptionsFromJson(JObject cfg)
        {
            PatternOptions.Clear();

            if (cfg == null) { FillOptionsFromTable(); return; }

            var opts = cfg["options"] as JArray;
            if (opts == null || opts.Count == 0)
            {
                // 沒 options：用預設名表
                FillOptionsFromTable();
                return;
            }

            // 用 JSON 覆蓋名表（若提供）
            for (int i = 0; i < opts.Count; i++)
            {
                var jo = opts[i] as JObject;
                if (jo == null) continue;

                int idx = ToInt(jo["index"], -1);
                if (idx < 0 || idx > 21) continue;

                var name = jo["name"] != null ? jo["name"].ToString() : null;
                if (!string.IsNullOrWhiteSpace(name))
                {
                    _nameByIndex[idx] = name.Trim();
                }
            }

            FillOptionsFromTable();
        }

        private void FillOptionsFromTable()
        {
            for (int i = 0; i <= 21; i++)
            {
                string name;
                if (!_nameByIndex.TryGetValue(i, out name) || string.IsNullOrWhiteSpace(name))
                {
                    name = i + ".Pattern";
                }
                PatternOptions.Add(new PatternOption { Index = i, Display = name });
            }
        }

        private void LoadOrderDefaultsFromJson(JObject cfg)
        {
            // 如果 JSON 有 orders.defaults: [0,1,2...]
            var ordersNode = cfg != null ? cfg["orders"] as JObject : null;
            if (ordersNode == null)
            {
                // 沒有 defaults：保持 0..Total-1
                for (int i = 0; i < Orders.Count; i++) Orders[i].SelectedIndex = i;
                return;
            }

            var defaults = ordersNode["defaults"] as JArray;
            if (defaults == null || defaults.Count == 0)
            {
                for (int i = 0; i < Orders.Count; i++) Orders[i].SelectedIndex = i;
                return;
            }

            for (int i = 0; i < Orders.Count; i++)
            {
                int def = i;
                if (i < defaults.Count) def = ToInt(defaults[i], i);
                def = Clamp(def, 0, 21);
                Orders[i].SelectedIndex = def;
            }
        }

        // =====================================================================
        // 5) 內部工具
        // =====================================================================
        private void ResizeOrders(int desired)
        {
            desired = Clamp(desired, TotalMin, TotalMax);

            while (Orders.Count < desired)
            {
                int no = Orders.Count + 1;      // 顯示用 1-based
                Orders.Add(new OrderSlot { DisplayNo = no, SelectedIndex = Orders.Count });
            }
            while (Orders.Count > desired)
            {
                Orders.RemoveAt(Orders.Count - 1);
            }

            // 重新編號顯示用序號
            for (int i = 0; i < Orders.Count; i++)
            {
                Orders[i].DisplayNo = i + 1;
            }
        }

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        private static int ToInt(JToken tok, int fallback)
        {
            if (tok == null) return fallback;
            try
            {
                int v;
                if (int.TryParse(tok.ToString(), out v)) return v;
                return fallback;
            }
            catch { return fallback; }
        }

        // =====================================================================
        // 6) 小型資料結構（C#7.3 風格）
        // =====================================================================
        public sealed class PatternOption : ViewModelBase
        {
            private int _index;
            private string _display;

            public int Index
            {
                get { return _index; }
                set { SetValue(ref _index, value); }
            }

            public string Display
            {
                get { return _display; }
                set { SetValue(ref _display, value); }
            }
        }

        public sealed class OrderSlot : ViewModelBase
        {
            private int _displayNo;
            private int _selectedIndex;

            public int DisplayNo
            {
                get { return _displayNo; }
                set { SetValue(ref _displayNo, value); }
            }

            // 綁定到 ComboBox.SelectedValue / SelectedItem (Index)
            public int SelectedIndex
            {
                get { return _selectedIndex; }
                set { SetValue(ref _selectedIndex, value); }
            }
        }
    }
}