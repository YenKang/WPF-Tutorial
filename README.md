private void EnsureSelection()
{
    if (MOptions.Count > 0 && !MOptions.Contains(M))
        M = MOptions[0]; // 保證 SelectedItem 有東西
    if (NOptions.Count > 0 && !NOptions.Contains(N))
        N = NOptions[0];
}

private void BuildOptions()
{
    MOptions.Clear(); NOptions.Clear();

    int mStart = Math.Min(_mMin, _mMax), mEnd = Math.Max(_mMin, _mMax);
    for (int v = mStart; v <= mEnd; v++) MOptions.Add(v);

    int nStart = Math.Min(_nMin, _nMax), nEnd = Math.Max(_nMin, _nMax);
    for (int v = nStart; v <= nEnd; v++) NOptions.Add(v);

    // 退避，避免空清單
    if (MOptions.Count == 0) for (int v = 2; v <= 40; v++) MOptions.Add(v);
    if (NOptions.Count == 0) for (int v = 2; v <= 40; v++) NOptions.Add(v);
}