/// <summary>
/// 確保 pattern index 在 PatternOptions 內有效
/// </summary>
private int CoercePatternIndex(int idx)
{
    if (PatternOptions == null || PatternOptions.Count == 0)
        return 0;

    // 若存在該 index 就回傳，否則退回第一個
    foreach (var opt in PatternOptions)
        if (opt.Index == idx)
            return idx;

    return PatternOptions[0].Index;
}