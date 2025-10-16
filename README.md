foreach (var node in _profile.Patterns)
{
    string iconUri = "pack://siteoforigin:,,,/Assets/Patterns/NT51365/" + node.icon;
    var item = new PatternItem
    {
        Index = node.index,
        Name = node.name,
        Icon = iconUri,
        RegControlType = node.regControlType,
        GrayLevelControl = node.grayLevelControl,
        // UI 專用欄位
        IsSelected = false,
        IsGrayLevelVisible = node.regControlType != null && node.regControlType.Contains("grayLevelControl")
    };
    Patterns.Add(item);
}