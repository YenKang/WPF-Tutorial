using System;
using System.ComponentModel;
using System.Globalization;
using System.Runtime.CompilerServices;

public class BistLevelViewModel : INotifyPropertyChanged
{
    // --- 三個 Digit（字串：High 0~3；Mid/Low 0~F） ---
    private string _highDigit = "3"; // 對應 D[9:8]
    public string HighDigit
    {
        get { return _highDigit; }
        set { if (_highDigit != value) { _highDigit = value; OnPropertyChanged(); RecomputeLevel(); } }
    }

    private string _midDigit = "F";  // 對應 D[7:4]
    public string MidDigit
    {
        get { return _midDigit; }
        set { if (_midDigit != value) { _midDigit = value; OnPropertyChanged(); RecomputeLevel(); } }
    }

    private string _lowDigit = "F";  // 對應 D[3:0]
    public string LowDigit
    {
        get { return _lowDigit; }
        set { if (_lowDigit != value) { _lowDigit = value; OnPropertyChanged(); RecomputeLevel(); } }
    }

    // 顯示用：例如 "3 FF"
    private string _hexDisplay = "3 FF";
    public string HexDisplay
    {
        get { return _hexDisplay; }
        private set { if (_hexDisplay != value) { _hexDisplay = value; OnPropertyChanged(); } }
    }

    // 顯示用：例如 "GrayLevel = 1023 (0x3FF)"
    private string _grayLevelDisplay = "GrayLevel = 1023 (0x3FF)";
    public string GrayLevelDisplay
    {
        get { return _grayLevelDisplay; }
        private set { if (_grayLevelDisplay != value) { _grayLevelDisplay = value; OnPropertyChanged(); } }
    }

    // 重新計算 10-bit 值 + 兩個顯示字串
    private void RecomputeLevel()
    {
        int h = ParseHex1(HighDigit); // 0..3
        int m = ParseHex1(MidDigit);  // 0..15
        int l = ParseHex1(LowDigit);  // 0..15

        // 組 10-bit：h 放到 bit[9:8]、m 放到 bit[7:4]、l 放到 bit[3:0]
        int value = (h << 8) | (m << 4) | l;

        // 顯示 "3 FF"
        HexDisplay = string.Format("{0:X1} {1:X1}{2:X1}", h, m, l);

        // 顯示 "GrayLevel = 1023 (0x3FF)"
        GrayLevelDisplay = "GrayLevel = " + value.ToString(CultureInfo.InvariantCulture)
                         + " (0x" + value.ToString("X3", CultureInfo.InvariantCulture) + ")";
    }

    private static int ParseHex1(string s)
    {
        if (string.IsNullOrEmpty(s)) return 0;
        return int.Parse(s.Trim(), NumberStyles.HexNumber, CultureInfo.InvariantCulture);
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void OnPropertyChanged([CallerMemberName] string name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
