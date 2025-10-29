// 1) 加上常數
private const int MMin = 2, MMax = 40, MDef = 5;
private const int NMin = 2, NMax = 40, NDef = 7;

// 2) 調整屬性：夾範圍
public int M { get => GetValue<int>(); set => SetValue(Clamp(value, MMin, MMax)); }
public int N { get => GetValue<int>(); set => SetValue(Clamp(value, NMin, NMax)); }

// 3) 重建選項時，確保 SelectedItem 仍合法
private void BuildOptions()
{
    MOptions.Clear();
    NOptions.Clear();
    for (int i = MMin; i <= MMax; i++) MOptions.Add(i);
    for (int i = NMin; i <= NMax; i++) NOptions.Add(i);

    CoerceMNSelection();  // ← 新增
}

private void CoerceMNSelection()
{
    if (M < MMin || M > MMax) M = MDef;
    if (N < NMin || N > NMax) N = NDef;

    // 再次保險：清單中真的有這個值
    if (!MOptions.Contains(M) && MOptions.Count > 0) M = MOptions[0];
    if (!NOptions.Contains(N) && NOptions.Count > 0) N = NOptions[0];
}

// 4) LoadFrom 結尾再保險一次
public void LoadFrom(JObject node)
{
    _cfg = node;

    // (A) 先設 default（原本的）
    if (node?["fields"] is JArray fields)
    {
        foreach (JObject f in fields)
        {
            string key = (string)f["key"];
            int def = (int?)f["default"] ?? 0;
            switch (key)
            {
                case "H_RES": HRes = def; break;
                case "V_RES": VRes = def; break;
                case "M":     M    = def; break;   // 若沒給，下面會補
                case "N":     N    = def; break;
            }
        }
    }

    // (B) 建清單後，把 M/N 壓回合法並確保選到
    BuildOptions();
    CoerceMNSelection();

    // (C) 嘗試回讀暫存器（不會改 M/N，只改 HRes/VRes）
    TryRefreshFromRegister();
}