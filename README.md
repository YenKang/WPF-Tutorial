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
        private bool _dirty  = false;               // 若你不綁 CanApply，可忽略它

        // JSON 範圍（由 JSON 載入）
        private int _mMin = 2, _mMax = 40;
        private int _nMin = 2, _nMax = 40;

        // 解析度 / 參數
        public int HRes { get { return GetValue<int>(); } set { SetValue(value); } }
        public int VRes { get { return GetValue<int>(); } set { SetValue(value); } }

        // 內部真實值（用於寫暫存器）
        public int M
        {
            get { return GetValue<int>(); }
            set
            {
                value = Clamp(value, _mMin, _mMax);
                SetValue(value);
                if (!_inInit) _dirty = true;
            }
        }
        public int N
        {
            get { return GetValue<int>(); }
            set
            {
                value = Clamp(value, _nMin, _nMax);
                SetValue(value);
                if (!_inInit) _dirty = true;
            }
        }

        // ComboBox 選單（依 JSON min/max 動態生成）
        private readonly ObservableCollection<int> _mOptions = new ObservableCollection<int>();
        private readonly ObservableCollection<int> _nOptions = new ObservableCollection<int>();
        public ObservableCollection<int> MOptions { get { return _mOptions; } }
        public ObservableCollection<int> NOptions { get { return _nOptions; } }

        // ComboBox 的 SelectedValue（直接映射到 M / N）
        public int M_Set { get { return M; } set { M = value; } }
        public int N_Set { get { return N; } set { N = value; } }

        // Command
        public ICommand ApplyCommand { get; private set; }

        public ChessBoardVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(Apply);
            // 保底值（JSON 載入後會覆蓋）
            HRes = 1920; VRes = 720; M = 5; N = 7;
            // 預先生成一份基本選單，避免 UI 初載空白
            BuildOptions();
        }

        // 由外部傳入 name="Chessboard" 節點
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
                                M = def;      // 會被 clamp 到範圍
                                break;
                            case "N":
                                if (min > 0) _nMin = min;
                                if (max > 0) _nMax = max;
                                N = def;      // 會被 clamp 到範圍
                                break;
                        }
                    }
                }

                // 依最新的 _mMin/_mMax/_nMin/_nMax 生成 ComboBox 清單
                BuildOptions();

                // 若 default 不在清單裡，調到範圍內第一個
                if (_mOptions.Count > 0 && !_mOptions.Contains(M)) M = _mOptions[0];
                if (_nOptions.Count > 0 && !_nOptions.Contains(N)) N = _nOptions[0];
            }
            finally
            {
                _inInit = false;
                _dirty  = false;
            }
        }

        private void BuildOptions()
        {
            _mOptions.Clear();
            _nOptions.Clear();

            int mStart = Math.Min(_mMin, _mMax);
            int mEnd   = Math.Max(_mMin, _mMax);
            for (int v = mStart; v <= mEnd; v++) _mOptions.Add(v);

            int nStart = Math.Min(_nMin, _nMax);
            int nEnd   = Math.Max(_nMin, _nMax);
            for (int v = nStart; v <= nEnd; v++) _nOptions.Add(v);
        }

        private void Apply()
        {
            if (_cfg == null) return;

            // 快照（避免執行中 UI 改值）
            int hRes = HRes, vRes = VRes, m = M, n = N;

            // 計算 Toggle 與 +1
            int th = (m > 0) ? hRes / m : 0;
            int tv = (n > 0) ? vRes / n : 0;

            // 依實際位寬夾限：H=低8+高5，V=低8+高4
            th = ClampToBitWidth(th, 8, 5);   // 0..0x1FFF
            tv = ClampToBitWidth(tv, 8, 4);   // 0..0x0FFF

            byte h_low   = (byte)(th & 0xFF);
            byte h_high  = (byte)((th >> 8) & 0x1F);  // 5 bits
            byte h_plus1 = (byte)((hRes - th * m) == 0 ? 0 : 1);

            byte v_low   = (byte)(tv & 0xFF);
            byte v_high  = (byte)((tv >> 8) & 0x0F);  // 4 bits
            byte v_plus1 = (byte)((vRes - tv * n) == 0 ? 0 : 1);

            // 1) H：003C/003D/0040
            RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", h_low);

            byte cur003D = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H");
            byte next003D = (byte)((cur003D & 0xE0) | (h_high & 0x1F));   // [4:0]
            if (next003D != cur003D)
                RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_H", next003D);

            RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", h_plus1);

            // 2) V：003E/003F/0041
            RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", v_low);

            byte cur003F = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H");
            byte next003F = (byte)((cur003F & 0xF0) | (v_high & 0x0F));   // [3:0]
            if (next003F != cur003F)
                RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_H", next003F);

            RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", v_plus1);

            // 可選：debug
            System.Diagnostics.Debug.WriteLine(
                $"[Chessboard] H={th} (0x{th:X})  L={h_low:X2} H={h_high:X1}  P1={h_plus1} | " +
                $"V={tv} (0x{tv:X})  L={v_low:X2} H={v_high:X1}  P1={v_plus1}"
            );

            _dirty = false; // 若你要用 CanApply 控制是否可按，這行會清掉 dirty
        }

        // ====== 小工具：無 .NET 依賴的 clamp ======
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

====

<GroupBox Header="Chessboard" Margin="0,8,0,0" DataContext="{Binding ChessBoardVM}">
  <StackPanel Margin="10">

    <!-- Resolution 顯示（維持原先寫法即可） -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Resolution:" Width="100"/>
      <TextBlock Text="{Binding HRes}" Width="60"/>
      <TextBlock Text="×" Margin="6,0"/>
      <TextBlock Text="{Binding VRes}" Width="60"/>
    </StackPanel>

    <!-- M / N：改用 ComboBox -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,10">
      <TextBlock Text="M (H split):" Width="100"/>
      <ComboBox Width="80"
                ItemsSource="{Binding MOptions}"
                SelectedValue="{Binding M_Set, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>

      <TextBlock Text="N (V split):" Width="100" Margin="24,0,0,0"/>
      <ComboBox Width="80"
                ItemsSource="{Binding NOptions}"
                SelectedValue="{Binding N_Set, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- Set 按鈕（可不綁 CanApply；若要綁，請確保你有 Raise 通知） -->
    <Button Content="Set" Width="120"
            Command="{Binding ApplyCommand}"/>
  </StackPanel>
</GroupBox>