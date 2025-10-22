// 先把 M/N 調整到「新範圍內」再 Clear/Add，避免 SelectedValue 瞬間掉成 null
private void BuildOptions()
{
    int mStart = Math.Min(_mMin, _mMax);
    int mEnd   = Math.Max(_mMin, _mMax);
    int nStart = Math.Min(_nMin, _nMax);
    int nEnd   = Math.Max(_nMin, _nMax);

    // 只在超出範圍時修正，不碰 JSON 已讀入的預設 5/7
    if (M < mStart || M > mEnd) M = mStart;
    if (N < nStart || N > nEnd) N = nStart;

    // ❶ 清單重建（不可 new，只能 Clear/Add）
    MOptions.Clear();
    for (int v = mStart; v <= mEnd; v++) MOptions.Add(v);

    NOptions.Clear();
    for (int v = nStart; v <= nEnd; v++) NOptions.Add(v);

    // 防呆：避免空清單
    if (MOptions.Count == 0) for (int v = 2; v <= 40; v++) MOptions.Add(v);
    if (NOptions.Count == 0) for (int v = 2; v <= 40; v++) NOptions.Add(v);

    // ❷ 最後保證選中（M/N 一定在清單中）
    EnsureSelection();
}

private void EnsureSelection()
{
    if (MOptions.Count > 0 && !MOptions.Contains(M)) M = MOptions[0];
    if (NOptions.Count > 0 && !NOptions.Contains(N)) N = NOptions[0];
}