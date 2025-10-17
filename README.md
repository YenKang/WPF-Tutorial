private void ApplyPatternToGroups(PatternItem p)
{
    var types = p?.RegControlType ?? new List<string>();

    // 原本：前景灰階
    IsGrayLevelVisible = types.Contains("grayLevelControl");
    if (IsGrayLevelVisible && p?.GrayLevelControl != null)
        GrayLevelVM.LoadFrom(p.GrayLevelControl);

    // 新增：背景灰階
    IsBgGrayLevelVisible = types.Contains("bgGrayLevelControl");
    if (IsBgGrayLevelVisible && p?.BgGrayLevelControl != null)
        BgGrayLevelVM.LoadFrom(p.BgGrayLevelControl);
}