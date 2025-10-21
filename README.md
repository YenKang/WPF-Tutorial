// ===== 在 AutoRunVM 類別內新增 =====
public FcntVm Fcnt1 { get; } = new FcntVm();
public FcntVm Fcnt2 { get; } = new FcntVm();

// 讀 JSON：在 AutoRunVM.LoadFrom(JObject autoRunControl) 結尾加入
var f1 = autoRunControl["fcnt1"] as JObject;
if (f1 != null) Fcnt1.LoadFrom(f1);

var f2 = autoRunControl["fcnt2"] as JObject;
if (f2 != null) Fcnt2.LoadFrom(f2);

// Apply：在 AutoRunVM.Apply() 末端（觸發前或後都行）加入
Fcnt1.ApplyWrite();
Fcnt2.ApplyWrite();


// ===== 放在 AutoRunVM 類別**裡面**的子類別 =====
public sealed class FcntVm : ViewModelBase
{
    // 目標與編碼資訊（由 JSON 注入）
    private string _lowTarget, _highTarget;
    private byte _highMask;           // 比方 0x03 或 0x0C（在該高位暫存器中的位元遮罩）
    private int  _highShift;          // 8 或 10（把 D2 左移幾 bit 寫進 high register）

    // D2: 2bits (0..3)；D1/D0: 4bits (0..15)
    public ObservableCollection<int> D2Options { get; } =
        new ObservableCollection<int>(Enumerable.Range(0, 4));
    public ObservableCollection<int> DnOptions { get; } =
        new ObservableCollection<int>(Enumerable.Range(0, 16));

    public int D2 { get => GetValue<int>(); set => SetValue(value); }
    public int D1 { get => GetValue<int>(); set => SetValue(value); }
    public int D0 { get => GetValue<int>(); set => SetValue(value); }

    // 從 JSON 載入：default 轉成 D2/D1/D0，並記住寫入目標
    public void LoadFrom(JObject jo)
    {
        if (jo == null) return;

        // default 10-bit 整數
        var defStr = (string)jo["default"];
        var def = ParseHexWord(defStr, 0x000);  // 0x3C / 0x1E 等

        D2 = (def >> 8) & 0x03; // 高 2 bits
        D1 = (def >> 4) & 0x0F;
        D0 = (def      ) & 0x0F;

        var tgt = jo["targets"] as JObject;
        _lowTarget  = (string)tgt?["low"];
        _highTarget = (string)tgt?["high"];
        _highMask   = ParseHexByte((string)tgt?["mask"], 0xFF);
        _highShift  = tgt?["shift"]?.Value<int>() ?? 0;
    }

    // 寫入
    public void ApplyWrite()
    {
        if (string.IsNullOrEmpty(_lowTarget))  return;  // 沒有配置就跳過

        // 低位元組：D1(高 nibble) + D0(低 nibble)
        byte low = (byte)(((D1 & 0x0F) << 4) | (D0 & 0x0F));
        RegMap.Write8(_lowTarget, low);

        // 高位 bits：把 D2 左移到對應欄位後，RMW 到 high register
        if (!string.IsNullOrEmpty(_highTarget) && _highMask != 0)
        {
            byte shifted = (byte)((D2 << _highShift) & _highMask);
            RegMap.Rmw(_highTarget, _highMask, shifted);
        }
    }

    // 小工具：16 進位字串 -> ushort / byte
    private static ushort ParseHexWord(string s, ushort fallback)
    {
        if (string.IsNullOrWhiteSpace(s)) return fallback;
        s = s.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
        ushort v;
        return UInt16.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out v) ? v : fallback;
    }
    private static byte ParseHexByte(string s, byte fallback)
    {
        if (string.IsNullOrWhiteSpace(s)) return fallback;
        s = s.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
        byte v;
        return Byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out v) ? v : fallback;
    }
}

=====

<!-- FCNT1 -->
<StackPanel Orientation="Horizontal" Margin="0,8,0,0">
  <TextBlock Text="FCNT1:" Width="100" VerticalAlignment="Center"/>

  <!-- D2 (bits9:8) -->
  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt1.D2Options}"
            SelectedItem="{Binding AutoRunVM.Fcnt1.D2, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Margin="0,0,8,0"/>

  <!-- D1 -->
  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt1.DnOptions}"
            SelectedItem="{Binding AutoRunVM.Fcnt1.D1, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Margin="0,0,8,0"/>

  <!-- D0 -->
  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt1.DnOptions}"
            SelectedItem="{Binding AutoRunVM.Fcnt1.D0, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}"/>
</StackPanel>

<!-- FCNT2 -->
<StackPanel Orientation="Horizontal" Margin="0,6,0,0">
  <TextBlock Text="FCNT2:" Width="100" VerticalAlignment="Center"/>

  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt2.D2Options}"
            SelectedItem="{Binding AutoRunVM.Fcnt2.D2, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Margin="0,0,8,0"/>

  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt2.DnOptions}"
            SelectedItem="{Binding AutoRunVM.Fcnt2.D1, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Margin="0,0,8,0"/>

  <ComboBox Width="60"
            ItemsSource="{Binding AutoRunVM.Fcnt2.DnOptions}"
            SelectedItem="{Binding AutoRunVM.Fcnt2.D0, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}"/>
</StackPanel>