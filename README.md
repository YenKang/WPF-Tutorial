using Newtonsoft.Json.Linq;
using System;
using System.Windows;
using System.Windows.Input;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;

        // JSON 讀出來的範圍 (預設值先給一組，LoadFrom 會覆蓋)
        private int _hResMin = 720, _hResMax = 6720;
        private int _vResMin = 136, _vResMax = 2560;
        private int _mMin    = 2,   _mMax    = 40;
        private int _nMin    = 2,   _nMax    = 40;

        // 從 JSON 讀到的暫存器名稱（H / V 分開）
        // 對應 chessH.registers / chessV.registers
        private string _regToggleH = "BIST_CHESSBOARD_H_TOGGLE_W";
        private string _regPlus1H  = "BIST_CHESSBOARD_H_PLUS1_BLKNUM";
        private string _regToggleV = "BIST_CHESSBOARD_V_TOGGLE_H";
        private string _regPlus1V  = "BIST_CHESSBOARD_V_PLUS1_BLKNUM";

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

        // M / N：實際 clamp 在 Apply() 再做一次
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
        /// 由外部注入的暫存器讀寫介面
        /// </summary>
        public PageBase.ICStructure.Comm.IRegisterReadWriteEx RegControl { get; set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            // 初始預設值（之後會被 JSON 覆蓋）
            HRes = 1920;
            VRes = 720;
            M    = 5;
            N    = 7;
        }

        /// <summary>
        /// 從 chessBoardControl 節點載入設定
        /// </summary>
        public void LoadFrom(JObject node)
        {
            _cfg = node;
            if (_cfg == null) return;

            try
            {
                // ---- fields: H_RES / V_RES / M / N ----
                if (node["fields"] is JArray fields)
                {
                    foreach (JObject f in fields)
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
                    foreach (JObject r in hRegs)
                    {
                        foreach (var p in r.Properties())
                        {
                            var name = p.Name;
                            var val = p.Value != null ? p.Value.ToString().Trim() : string.Empty;
                            if (string.IsNullOrEmpty(val)) continue;

                            // 最終 JSON：Toggle_Width / H_PLUS1
                            switch (name)
                            {
                                case "Toggle_Width": _regToggleH = val; break;
                                case "H_PLUS1":      _regPlus1H  = val; break;
                            }
                        }
                    }
                }

                // ---- chessV.registers ----
                var chessV = node["chessV"] as JObject;
                if (chessV != null && chessV["registers"] is JArray vRegs)
                {
                    foreach (JObject r in vRegs)
                    {
                        foreach (var p in r.Properties())
                        {
                            var name = p.Name;
                            var val = p.Value != null ? p.Value.ToString().Trim() : string.Empty;
                            if (string.IsNullOrEmpty(val)) continue;

                            // 最終 JSON：Toggle_Height / V_PLUS1
                            switch (name)
                            {
                                case "Toggle_Height": _regToggleV = val; break;
                                case "V_PLUS1":       _regPlus1V  = val; break;
                            }
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Load json 發生錯誤: " + ex);
            }
        }

        /// <summary>
        /// 按下 Set：
        /// 1. clamp H/V/M/N
        /// 2. 用「選項 A」公式計算 th/tv/hPlus/vPlus
        /// 3. 透過 SetRegisterByName 寫入暫存器
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

            // 邊界保護
            HRes = Clamp(HRes, _hResMin, _hResMax);
            VRes = Clamp(VRes, _vResMin, _vResMax);
            M    = Clamp(M, _mMin, _mMax);
            N    = Clamp(N, _nMin, _nMax);

            int hRes = HRes;
            int vRes = VRes;

            int m = M <= 0 ? 1 : M;
            int n = N <= 0 ? 1 : N;

            // 基本寬度 / 高度（整除部分）
            int th = hRes / m;   // 每一格的水平寬度
            int tv = vRes / n;   // 每一格的垂直高度

            // "+1 分配數" = 餘數有幾個格子是 (basic + 1)
            int hPlus = hRes - th * m;  // 0..(m-1)
            int vPlus = vRes - tv * n;  // 0..(n-1)

            if (th    < 0) th    = 0;
            if (tv    < 0) tv    = 0;
            if (hPlus < 0) hPlus = 0;
            if (vPlus < 0) vPlus = 0;

            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] HRes={hRes}, VRes={vRes}, M={m}, N={n}, " +
                $"th={th}, tv={tv}, hPlus={hPlus}, vPlus={vPlus}, " +
                $"regH_Toggle={_regToggleH}, regH_Plus1={_regPlus1H}, " +
                $"regV_Toggle={_regToggleV}, regV_Plus1={_regPlus1V}");

            // 直接寫入暫存器
            RegControl.SetRegisterByName(_regToggleH, (uint)th);
            RegControl.SetRegisterByName(_regPlus1H,  (uint)hPlus);

            RegControl.SetRegisterByName(_regToggleV, (uint)tv);
            RegControl.SetRegisterByName(_regPlus1V,  (uint)vPlus);
        }
    }
}