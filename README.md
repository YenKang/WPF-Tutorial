using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        // === 欄位限制 ===
        private const int HResMin = 720,  HResMax = 6720;
        private const int VResMin = 136,  VResMax = 2560;

        // === 屬性 ===
        public int HRes { get => GetValue<int>(); set => SetValue(Clamp(value, HResMin, HResMax)); }
        public int VRes { get => GetValue<int>(); set => SetValue(Clamp(value, VResMin, VResMax)); }

        public ObservableCollection<int> MOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> NOptions { get; } = new ObservableCollection<int>();
        public int M { get => GetValue<int>(); set => SetValue(value); }
        public int N { get => GetValue<int>(); set => SetValue(value); }

        private JObject _cfg;

        private static int Clamp(int v, int min, int max)
            => v < min ? min : (v > max ? max : v);

        // === 初始化 ===
        public void LoadFrom(JObject node)
        {
            _cfg = node;

            // 1️⃣ 從 JSON 讀取預設值
            if (node?["fields"] is JArray fields)
            {
                foreach (JObject f in fields)
                {
                    string key = (string)f["key"];
                    int def = (int?)f["default"] ?? 0;

                    switch (key)
                    {
                        case "H_RES": HRes = def; break;
                        case "V_RES": VRes = def; break;
                        case "M":     M    = def; break;
                        case "N":     N    = def; break;
                    }
                }
            }

            // 2️⃣ 建立 ComboBox 可選項
            BuildOptions();

            // 3️⃣ 嘗試從暫存器回讀實際值覆蓋
            TryRefreshFromRegister();
        }

        private void BuildOptions()
        {
            MOptions.Clear();
            NOptions.Clear();
            for (int i = 2; i <= 40; i++)
            {
                MOptions.Add(i);
                NOptions.Add(i);
            }
        }

        // === Set 按鈕 ===
        public void Apply()
        {
            try
            {
                // 實際寫入暫存器邏輯
                WriteToRegisters();

                // 寫入完 → 立刻回讀確認同步
                TryRefreshFromRegister();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] Apply failed: " + ex.Message);
            }
        }

        // === 回讀暫存器 ===
        public void RefreshFromRegister()
        {
            // H 軸
            int hLow   = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_L");    // 003Ch
            int hHigh  = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H") & 0x1F; // 003Dh
            int hPlus1 = RegMap.Read8("BIST_CHESSBOARD_H_PLUS1_BLKNUM") & 0x01; // 0040h bit0
            int th     = (hHigh << 8) | (hLow & 0xFF);

            // V 軸
            int vLow   = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_L");    // 003Eh
            int vHigh  = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H") & 0x0F; // 003Fh
            int vPlus1 = RegMap.Read8("BIST_CHESSBOARD_V_PLUS1_BLKNUM") & 0x01; // 0041h bit0
            int tv     = (vHigh << 8) | (vLow & 0xFF);

            if (M > 0)
                HRes = Clamp(th * M + (hPlus1 != 0 ? 1 : 0), HResMin, HResMax);
            if (N > 0)
                VRes = Clamp(tv * N + (vPlus1 != 0 ? 1 : 0), VResMin, VResMax);

            System.Diagnostics.Debug.WriteLine($"[Chessboard] RefreshFromRegister → HRes={HRes}, VRes={VRes}");
        }

        private void TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] Skip refresh: " + ex.Message);
            }
        }

        // === 寫入暫存器 ===
        private void WriteToRegisters()
        {
            // 這裡用你的現有邏輯（範例）
            // 例如：
            int th = HRes / M;
            int hPlus1 = (HRes % M) != 0 ? 1 : 0;

            int tv = VRes / N;
            int vPlus1 = (VRes % N) != 0 ? 1 : 0;

            // H
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", th & 0xFF);
            RegMap.Rmw("BIST_CHESSBOARD_H_TOGGLE_W_H", (th >> 8) & 0x1F, 0x1F, 0);
            RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", hPlus1);

            // V
            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", tv & 0xFF);
            RegMap.Rmw("BIST_CHESSBOARD_V_TOGGLE_W_H", (tv >> 8) & 0x0F, 0x0F, 0);
            RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", vPlus1);

            System.Diagnostics.Debug.WriteLine($"[Chessboard] Apply success: HRes={HRes}, VRes={VRes}, M={M}, N={N}");
        }
    }
}