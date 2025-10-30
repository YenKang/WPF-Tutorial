using Newtonsoft.Json.Linq;
using System;
using System.Collections.ObjectModel;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : Utility.MVVM.ViewModelBase
    {
        private JObject _cfg;
        private bool _preferHw = false;

        // === 解析度邊界 ===
        private const int HResMin = 720, HResMax = 6720;
        private const int VResMin = 136, VResMax = 2560;

        private const int MMin = 2, MMax = 40;
        private const int NMin = 2, NMax = 40;

        private static int Clamp(int v, int min, int max) => v < min ? min : (v > max ? max : v);

        // === 屬性 ===
        public int HRes { get => GetValue<int>(); set => SetValue(Clamp(value, HResMin, HResMax)); }
        public int VRes { get => GetValue<int>(); set => SetValue(Clamp(value, VResMin, VResMax)); }

        public int M { get => GetValue<int>(); set => SetValue(Clamp(value, MMin, MMax)); }
        public int N { get => GetValue<int>(); set => SetValue(Clamp(value, NMin, NMax)); }

        public ObservableCollection<int> MOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> NOptions { get; } = new ObservableCollection<int>();

        public RelayCommand ApplyCommand { get; }

        public ChessBoardVM()
        {
            ApplyCommand = new RelayCommand(Apply);
            BuildOptions();
        }

        private void BuildOptions()
        {
            MOptions.Clear();
            NOptions.Clear();
            for (int i = MMin; i <= MMax; i++) MOptions.Add(i);
            for (int j = NMin; j <= NMax; j++) NOptions.Add(j);
        }

        // === 載入 JSON ===
        public void LoadFrom(JObject node)
        {
            _cfg = node;
            try
            {
                if (node["fields"] is JArray fields)
                {
                    foreach (JObject f in fields)
                    {
                        string key = (string)f["key"];
                        int def = (int?)f["default"] ?? 0;
                        switch (key)
                        {
                            case "H_RES": HRes = def; break;
                            case "V_RES": VRes = def; break;
                            case "M": M = def; break;
                            case "N": N = def; break;
                        }
                    }
                }

                // ✅ 若已經 Set 過，下次切圖時用硬體真值覆蓋
                if (_preferHw)
                    TryRefreshFromRegister();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] LoadFrom failed: " + ex.Message);
            }
        }

        // === Set 按鈕 ===
        public void Apply()
        {
            try
            {
                WriteToRegisters();
                _preferHw = true;         // ✅ 寫入後，啟用硬體優先模式
                TryRefreshFromRegister(); // ✅ 寫完立刻同步 UI
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] Apply failed: " + ex.Message);
            }
        }

        // === 寫入暫存器 ===
        private void WriteToRegisters()
        {
            // 計算每格寬度 (toggle width)
            int th = HRes / M;
            int tv = VRes / N;

            // --- 寫入 Toggle Width (H/V) ---
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", th & 0xFF);
            RegMap.Rmw("BIST_CHESSBOARD_H_TOGGLE_W_H", (th >> 8) & 0x1F, 0x1F, 0);

            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", tv & 0xFF);
            RegMap.Rmw("BIST_CHESSBOARD_V_TOGGLE_W_H", (tv >> 8) & 0x0F, 0x0F, 0);

            // --- 寫入解析度 (H/V) ---
            RegMap.Write8("HRES_Low8",  (byte)(HRes & 0xFF));
            RegMap.Rmw("HRES_High5", (byte)((HRes >> 8) & 0x1F), 0x1F, 0);

            RegMap.Write8("VRES_Low8",  (byte)(VRes & 0xFF));
            RegMap.Rmw("VRES_High4", (byte)((VRes >> 8) & 0x0F), 0x0F, 0);

            System.Diagnostics.Debug.WriteLine($"[Chessboard] Write OK: HRes={HRes}, VRes={VRes}, M={M}, N={N}, th={th}, tv={tv}");
        }

        // === 從暫存器回讀 ===
        public void RefreshFromRegister()
        {
            try
            {
                // 1️⃣ 讀取 toggle width (H/V)
                int hLow  = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_L");
                int hHigh = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H") & 0x1F;
                int th    = (hHigh << 8) | hLow;

                int vLow  = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_L");
                int vHigh = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H") & 0x0F;
                int tv    = (vHigh << 8) | vLow;

                // 2️⃣ 讀取解析度
                int hResLow  = RegMap.Read8("HRES_Low8");
                int vResLow  = RegMap.Read8("VRES_Low8");
                int hResHigh = RegMap.Read8("HRES_High5") & 0x1F;
                int vResHigh = RegMap.Read8("VRES_High4") & 0x0F;

                int hRes = ((hResHigh << 8) | hResLow) & 0x1FFF;
                int vRes = ((vResHigh << 8) | vResLow) & 0x0FFF;

                HRes = Clamp(hRes, HResMin, HResMax);
                VRes = Clamp(vRes, VResMin, VResMax);

                // 3️⃣ 反推 M/N
                if (th > 0) M = Clamp(HRes / th, MMin, MMax);
                if (tv > 0) N = Clamp(VRes / tv, NMin, NMax);

                System.Diagnostics.Debug.WriteLine($"[Chessboard] RefreshFromRegister → HRes={HRes}, VRes={VRes}, th={th}, tv={tv}, M={M}, N={N}");
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] RefreshFromRegister failed: " + ex.Message);
            }
        }

        private void TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); } catch { /* 安全略過 */ }
        }
    }
}