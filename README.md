// 1) VM 實例（共用一個）
public GrayLevelVM GrayLevelVM { get; } = new GrayLevelVM();

// 2) 可見旗標（給 XAML 綁 Visibility）
private bool _isGrayLevelVisible;
public bool IsGrayLevelVisible
{
    get => _isGrayLevelVisible;
    set { if (_isGrayLevelVisible == value) return; _isGrayLevelVisible = value; OnPropertyChanged(); }
}

// 3) Pattern 切換時套用（SelectedPattern 的 setter 裡呼叫）
private void ApplyPatternToGroups(PatternItem p)
{
    var types = (p?.RegControlType) ?? new List<string>();
    IsGrayLevelVisible = types.Contains("grayLevelControl");

    if (IsGrayLevelVisible && p.GrayLevelControl != null)
        GrayLevelVM.LoadFrom(p.GrayLevelControl);   // ← 把 JSON 區塊餵進去
}