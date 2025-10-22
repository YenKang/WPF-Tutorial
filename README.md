// 先加 using
using System.Collections.ObjectModel;

// 這兩個放在 ChessBoardVM 類別內
private readonly ObservableCollection<int> _mOptions = new ObservableCollection<int>();
private readonly ObservableCollection<int> _nOptions = new ObservableCollection<int>();

public ObservableCollection<int> MOptions { get { return _mOptions; } }
public ObservableCollection<int> NOptions { get { return _nOptions; } }

// 供 ComboBox 綁定的 SelectedValue（同步到原本的 M/N）
public int M_Set
{
    get { return M; }
    set { M = value; }  // 仍走你原本的 M setter（含 clamp 與 _dirty）
}
public int N_Set
{
    get { return N; }
    set { N = value; }
}

// 在 LoadFrom(JObject node) 裡面，讀到 M/N 的 min/max 之後「建立清單」
private void RebuildOptions()
{
    _mOptions.Clear();
    _nOptions.Clear();

    int mStart = (_mMin <= _mMax) ? _mMin : _mMax;
    int mEnd   = (_mMin <= _mMax) ? _mMax : _mMin;
    for (int v = mStart; v <= mEnd; v++) _mOptions.Add(v);

    int nStart = (_nMin <= _nMax) ? _nMin : _nMax;
    int nEnd   = (_nMin <= _nMax) ? _nMax : _nMin;
    for (int v = nStart; v <= nEnd; v++) _nOptions.Add(v);

    // 讓 SelectedValue 有值（若 default 不在範圍，會被 M/N 的 clamp 自動夾回）
    if (_mOptions.Count > 0 && ! _mOptions.Contains(M)) M = _mOptions[0];
    if (_nOptions.Count > 0 && ! _nOptions.Contains(N)) N = _nOptions[0];
}