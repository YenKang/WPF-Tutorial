using Newtonsoft.Json.Linq;
using System;
using System.Collections.ObjectModel;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;

        // JSON 參數 (由 fields 覆蓋)
        private int _hResMin = 720, _hResMax = 6720;
        private int _vResMin = 136, _vResMax = 2560;
        private int _mMin    = 2,   _mMax    = 40;
        private int _nMin    = 2,   _nMax    = 40;

        // registers from JSON (chessH / chessV)
        private string _regToggleHeight = "BIST_CHESSBOARD_H_TOGGLE_W";
        private string _regHeightPlus1  = "BIST_CHESSBOARD_H_PLUS1_BLKNUM";
        private string _regToggleWidth  = "BIST_CHESSBOARD_V_TOGGLE_H";
        private string _regWidthPlus1   = "BIST_CHESSBOARD_V_PLUS1_BLKNUM";

        // 給 ComboBox 的項目
        public ObservableCollection<int> MOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> NOptions { get; } = new ObservableCollection<int>();

        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }

        // ========= 屬性 =========

        public int HRes
        {
            get { return GetValue<int>(); }
            set { SetValue(Clamp(value, _hResMin, _hResMax)); }
        }

        public int VRes
        {
            get { return GetValue<int>(); }
            set { SetValue(Clamp(value, _vResMin, _vResMax)); }
        }

        // M / N 仍然有 clamp（在 Apply 時做一次保護）
        public int M
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public int N
        {
            get { return GetValue<int>(); }
            set { SetValue(value); }
        }

        public ICommand ApplyCommand { get; }

        /// <summary>
        /// 對應 SetRegisterByName 的介面（外部注入）
        /// </summary>
        public PageBase.ICStructure.Comm.IRegisterReadWriteEx RegControl { get; set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            // 先給預設值（之後會被 JSON 覆蓋）
            HRes = 1920;
            VRes = 720;
            M = 5;
            N = 7;

            BuildOptions();
        }

        /// <summary>
        /// 從 "chessBoardControl" 節點載入設定
        /// node 結構：
        /// {
        ///   "fields": [ { key, default, min, max }, ... ],
        ///   "chessH": { "registers": [ { "Toggle_Height": "..." }, { "Height_PLUS1": "..." } ] },
        ///   "chessV": { "registers": [ { "Toggle_Width": "..." }, { "Width_PLUS1": "..." } ] }
        /// }
        /// </summary>
        public void LoadFrom(JObject node)
        {
            _cfg = node;
            if (_cfg == null) return;

            try
            {
                // ---- fields ----
                if (node["fields"] is JArray fields)
                {
                    foreach (JObject f in fields.Cast<JObject>())
                    {
                        string key = (string)f["key"];
                        int def = (int?)f["default"] ?? 0;
                        int min = (int?)f["min"] ?? 0;
                        int max = (int?)f["max"] ?? 0;

                        switch (key)
                        {
                            case "H_RES":
                                if (min > 0) _hResMin = min;
                                if (max > 0) _hResMax = max;
                                HRes = def;
                                break;

                            case "V_RES":
                                if (min > 0) _vResMin = min;
                                if (max > 0) _vResMax = max;
                                VRes = def;
                                break;

                            case "M":
                                if (min > 0) _mMin = min;
                                if (max > 0) _mMax = max;
                                M = def;
                                break;

                            case "N":
                                if (min > 0) _nMin = min;
                                if (max > 0) _nMax = max;
                                N = def;
                                break;
                        }
                    }
                }

                // ---- chessH.registers ----
                var chessH = node["chessH"] as JObject;
                if (chessH != null && chessH["registers"] is JArray hRegs)
                {
                    foreach (JObject r in hRegs.Cast<JObject>())
                    {
                        foreach (var p in r.Properties())
                        {
                            var name = p.Name;
                            var val = p.Value != null ? p.Value.ToString().Trim() : string.Empty;
                            if (string.IsNullOrEmpty(val)) continue;

                            switch (name)
                            {
                                case "Toggle_Height": _regToggleHeight = val; break;
                                case "Height_PLUS1":  _regHeightPlus1  = val; break;
                            }
                        }
                    }
                }

                // ---- chessV.registers ----
                var chessV = node["chessV"] as JObject;
                if (chessV != null && chessV["registers"] is JArray vRegs)
                {
                    foreach (JObject r in vRegs.Cast<JObject>())
                    {
                        foreach (var p in r.Properties())
                        {
                            var name = p.Name;
                            var val = p.Value != null ? p.Value.ToString().Trim() : string.Empty;
                            if (string.IsNullOrEmpty(val)) continue;

                            switch (name)
                            {
                                case "Toggle_Width": _regToggleWidth = val; break;
                                case "Width_PLUS1":  _regWidthPlus1  = val; break;
                            }
                        }
                    }
                }

                BuildOptions();
            }
            catch (Exception ex)
            {
                MessageBox.Show("Load json 發生錯誤: " + ex);
            }
        }

        /// <summary>
        /// 產生 M / N ComboBox 選項
        /// </summary>
        private void BuildOptions()
        {
            int mStart = Math.Min(_mMin, _mMax);
            int mEnd   = Math.Max(_mMin, _mMax);
            int nStart = Math.Min(_nMin, _nMax);
            int nEnd   = Math.Max(_nMin, _nMax);

            // 若超出範圍，拉回 JSON 指定範圍
            if (M < mStart || M > mEnd) M = mStart;
            if (N < nStart || N > nEnd) N = nStart;

            MOptions.Clear();
            for (int v = mStart; v <= mEnd; v++)
                MOptions.Add(v);

            NOptions.Clear();
            for (int v = nStart; v <= nEnd; v++)
                NOptions.Add(v);

            // 防呆：避免空清單
            if (MOptions.Count == 0)
                for (int v = 2; v <= 40; v++) MOptions.Add(v);
            if (NOptions.Count == 0)
                for (int v = 2; v <= 40; v++) NOptions.Add(v);
        }

        /// <summary>
        /// 按下 Set 之後：
        /// 1. clamp H/V/M/N
        /// 2. 計算 toggle / plus1
        /// 3. 透過 RegControl.SetRegisterByName 寫入四個暫存器
        /// </summary>
        private void Apply()
        {
            if (_cfg == null)
            {
                MessageBox.Show("Chessboard JSON 尚未載入。");
                return;
            }

            if (RegControl == null)
            {
                MessageBox.Show("RegControl 尚未注入。");
                return;
            }

            // clamp 一次
            HRes = Clamp(HRes, _hResMin, _hResMax);
            VRes = Clamp(VRes, _vResMin, _vResMax);
            M    = Clamp(M, _mMin, _mMax);
            N    = Clamp(N, _nMin, _nMax);

            int hRes = HRes;
            int vRes = VRes;
            int m    = M;
            int n    = N;

            if (m <= 0 || n <= 0)
            {
                MessageBox.Show("M / N 不可為 0。");
                return;
            }

            // 舊公式：toggle = (Res / 分割數) - 1, plus1 = toggle + 1
            int toggleH = (hRes / m) - 1;
            int plus1H  = toggleH + 1;

            int toggleV = (vRes / n) - 1;
            int plus1V  = toggleV + 1;

            if (toggleH < 0) toggleH = 0;
            if (toggleV < 0) toggleV = 0;
            if (plus1H  < 0) plus1H  = 0;
            if (plus1V  < 0) plus1V  = 0;

            // Debug trace
            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] HRes={hRes}, VRes={vRes}, M={m}, N={n}, " +
                $"toggleH={toggleH}, plus1H={plus1H}, toggleV={toggleV}, plus1V={plus1V}");

            // 直接用 SetRegisterByName 寫入暫存器
            RegControl.SetRegisterByName(_regToggleHeight, (uint)toggleH);
            RegControl.SetRegisterByName(_regHeightPlus1,  (uint)plus1H);

            RegControl.SetRegisterByName(_regToggleWidth,  (uint)toggleV);
            RegControl.SetRegisterByName(_regWidthPlus1,   (uint)plus1V);
        }
    }
}