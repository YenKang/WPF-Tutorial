using System;
using System.Collections.ObjectModel;
using Newtonsoft.Json.Linq;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public class ChessBoardVM : ViewModelBase
    {
        private JObject _cfg;
        private bool _inInit = false;

        // JSON 範圍（M/N）
        private int _mMin = 2, _mMax = 40;
        private int _nMin = 2, _nMax = 40;

        // 給 ComboBox 的項目
        public ObservableCollection<int> MOptions { get; } = new ObservableCollection<int>();
        public ObservableCollection<int> NOptions { get; } = new ObservableCollection<int>();

        // 解析度
        public int HRes { get { return GetValue<int>(); } set { SetValue(value); } }
        public int VRes { get { return GetValue<int>(); } set { SetValue(value); } }

        // 只保留 M / N（含 clamp）
        public int M
        {
            get { return GetValue<int>(); }
            set { SetValue(Clamp(value, _mMin, _mMax)); }
        }
        public int N
        {
            get { return GetValue<int>(); }
            set { SetValue(Clamp(value, _nMin, _nMax)); }
        }

        public ICommand ApplyCommand { get; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);

            // 先給預設值（JSON 會覆蓋）
            HRes = 1920; VRes = 720; M = 5; N = 7;

            // 產生選單並保證選中
            BuildOptions();
            EnsureSelection();
        }

        // 從 name="Chessboard" 節點載入
        public void LoadFrom(JObject node)
        {
            _inInit = true;
            try
            {
                _cfg = node;

                var fields = node["fields"] as JArray;
                if (fields != null)
                {
                    foreach (JObject f in fields)
                    {
                        string key = (string)f["key"];
                        int def = (int?)f["default"] ?? 0;
                        int min = (int?)f["min"] ?? 0;
                        int max = (int?)f["max"] ?? 0;

                        switch (key)
                        {
                            case "H_RES": HRes = def; break;
                            case "V_RES": VRes = def; break;
                            case "M":
                                if (min > 0) _mMin = min;
                                if (max > 0) _mMax = max;
                                M = def; // clamp 到範圍內
                                break;
                            case "N":
                                if (min > 0) _nMin = min;
                                if (max > 0) _nMax = max;
                                N = def; // clamp 到範圍內
                                break;
                        }
                    }
                }

                BuildOptions();     // 依最新範圍重建 2..40（或 JSON 範圍）
                EnsureSelection();  // 確保 SelectedItem 一定有效
            }
            finally { _inInit = false; }
        }

        private void BuildOptions()
        {
            MOptions.Clear();
            NOptions.Clear();

            int mStart = Math.Min(_mMin, _mMax), mEnd = Math.Max(_mMin, _mMax);
            for (int v = mStart; v <= mEnd; v++) MOptions.Add(v);

            int nStart = Math.Min(_nMin, _nMax), nEnd = Math.Max(_nMin, _nMax);
            for (int v = nStart; v <= nEnd; v++) NOptions.Add(v);

            // 防呆：若 JSON 範圍異常，退回 2..40
            if (MOptions.Count == 0) for (int v = 2; v <= 40; v++) MOptions.Add(v);
            if (NOptions.Count == 0) for (int v = 2; v <= 40; v++) NOptions.Add(v);
        }

        // 關鍵：在 ItemsSource 建好後，保證 SelectedItem(M/N) 一定在清單裡
        private void EnsureSelection()
        {
            if (MOptions.Count > 0 && !MOptions.Contains(M)) M = MOptions[0];
            if (NOptions.Count > 0 && !NOptions.Contains(N)) N = NOptions[0];
        }

        private void Apply()
        {
            if (_cfg == null) return;

            // 快照
            int hRes = HRes, vRes = VRes, m = M, n = N;

            // 計算 toggle 與 plus1
            int th = (m > 0) ? hRes / m : 0;
            int tv = (n > 0) ? vRes / n : 0;

            // 依暫存器位寬夾限：H=低8+高5，V=低8+高4
            th = ClampToBitWidth(th, 8, 5);   // 0..0x1FFF
            tv = ClampToBitWidth(tv, 8, 4);   // 0..0x0FFF

            byte h_low   = (byte)(th & 0xFF);
            byte h_high5 = (byte)((th >> 8) & 0x1F);  // 5 bits
            byte h_plus1 = (byte)((hRes - th * m) == 0 ? 0 : 1);

            byte v_low   = (byte)(tv & 0xFF);
            byte v_high4 = (byte)((tv >> 8) & 0x0F);  // 4 bits
            byte v_plus1 = (byte)((vRes - tv * n) == 0 ? 0 : 1);

            // 寫入暫存器 —— H（003C/003D/0040）
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", h_low);
            byte cur003D = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H");
            byte next003D = (byte)((cur003D & 0xE0) | (h_high5 & 0x1F)); // [4:0]
            if (next003D != cur003D) RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_H", next003D);
            RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", h_plus1);

            // 寫入暫存器 —— V（003E/003F/0041）
            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", v_low);
            byte cur003F = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H");
            byte next003F = (byte)((cur003F & 0xF0) | (v_high4 & 0x0F)); // [3:0]
            if (next003F != cur003F) RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_H", next003F);
            RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", v_plus1);
        }

        // —— 小工具：不依賴 Math.Clamp —— 
        private static int Clamp(int v, int min, int max)
        {
            if (v < min) return min;
            if (v > max) return max;
            return v;
        }
        private static int ClampToBitWidth(int value, int lowBits, int highBits)
        {
            if (value < 0) return 0;
            int total = lowBits + highBits;
            int max = (1 << total) - 1;
            return (value > max) ? max : value;
        }
    }
}