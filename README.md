using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        // === 常數：解析度邊界 ===
        private const int HResMin = 720,  HResMax = 6720;
        private const int VResMin = 136,  VResMax = 2560;

        // === 屬性 ===
        public int HRes
        {
            get => GetValue<int>();
            set => SetValue(Clamp(value, HResMin, HResMax));
        }

        public int VRes
        {
            get => GetValue<int>();
            set => SetValue(Clamp(value, VResMin, VResMax));
        }

        public ObservableCollection<int> MOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> NOptions { get; } = new ObservableCollection<int>();

        public int M
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        public int N
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        private JObject _cfg;

        // === Command ===
        public ICommand ApplyCommand { get; }

        // === 建構子 ===
        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
        }

        // === Clamp ===
        private static int Clamp(int v, int min, int max)
            => v < min ? min : (v > max ? max : v);

        // === 初始化：從 JSON 載入 ===
        public void LoadFrom(JObject node)
        {
            _cfg = node;

            try
            {
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
                            case "M":     M = def; break;
                            case "N":     N = def; break;
                        }
                    }
                }

                // 2️⃣ 建立 ComboBox 選項
                BuildOptions();

                // 3️⃣ 初次載入後回讀暫存器（同步真實狀態）
                TryRefreshFromRegister();
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] LoadFrom failed: " + ex.Message);
            }
        }

        // === ComboBox 選項建立 ===
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

        // === Command: Set 按鈕 ===
        private void Apply()
        {
            try
            {
                WriteToRegisters();
                TryRefreshFromRegister(); // 寫完立刻同步
            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] Apply failed: " + ex.Message);
            }
        }

        // === 暫存器寫入 ===
        private void WriteToRegisters()
        {
            if (M <= 0 || N <= 0)
                return; // 避免除以零

            // 計算平均方塊寬高
            int th = HRes / M;
            int hPlus1 = (HRes % M) != 0 ? 1 : 0;

            int tv = VRes / N;
            int vPlus1 = (VRes % N) != 0 ? 1 : 0;

            // === H 軸 ===
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", (byte)(th & 0xFF));

            RegMap.Rmw8("BIST_CHESSBOARD_H_TOGGLE_W_H",
                        (byte)0x1F,                       // [4:0]
                        (byte)((th >> 8) & 0x1F));

            RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", (byte)(hPlus1 & 0x01));

            // === V 軸 ===
            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", (byte)(tv & 0xFF));

            RegMap.Rmw8("BIST_CHESSBOARD_V_TOGGLE_W_H",
                        (byte)0x0F,                       // [3:0]
                        (byte)((tv >> 8) & 0x0F));

            RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", (byte)(vPlus1 & 0x01));

            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] Apply success: HRes={HRes}, VRes={VRes}, M={M}, N={N}");
        }

        // === 暫存器回讀 ===
        public void RefreshFromRegister()
        {
            // H 軸
            int hLow   = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_L");           // 003Ch
            int hHigh  = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H") & 0x1F;    // 003Dh
            int hPlus1 = RegMap.Read8("BIST_CHESSBOARD_H_PLUS1_BLKNUM") & 0x01;  // 0040h
            int th     = (hHigh << 8) | hLow;

            // V 軸
            int vLow   = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_L");           // 003Eh
            int vHigh  = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H") & 0x0F;    // 003Fh
            int vPlus1 = RegMap.Read8("BIST_CHESSBOARD_V_PLUS1_BLKNUM") & 0x01;  // 0041h
            int tv     = (vHigh << 8) | vLow;

            // 以 M/N 計算實際解析度
            if (M > 0)
                HRes = Clamp(th * M + (hPlus1 != 0 ? 1 : 0), HResMin, HResMax);

            if (N > 0)
                VRes = Clamp(tv * N + (vPlus1 != 0 ? 1 : 0), VResMin, VResMax);

            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] RefreshFromRegister OK → HRes={HRes}, VRes={VRes}");
        }

        private void TryRefreshFromRegister()
        {
            try { RefreshFromRegister(); }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine("[Chessboard] Skip refresh: " + ex.Message);
            }
        }
    }
}