public void LoadFrom(JObject autoRunControl)
{
    _cfg = autoRunControl;

    // 1) 讀 Title（不論是否 _preferHw 都可更新）
    Title = (_cfg != null && _cfg["title"] != null) ? _cfg["title"].ToString() : "Auto Run";

    // 2) 讀 options（index↔name）→ UI 顯示與 log 對照表
    PatternOptions.Clear();
    _indexToName.Clear();
    if (_cfg != null && _cfg["options"] is JArray opts)
    {
        foreach (JObject o in opts)
        {
            int idx = (int?)o["index"] ?? 0;
            string name = (string)o["name"] ?? idx.ToString();
            PatternOptions.Add(new PatternOption { Index = idx, Display = name });
            if (!_indexToName.ContainsKey(idx)) _indexToName.Add(idx, name);
        }
    }
    else
    {
        // fallback：至少補 0..63，避免空選單
        for (int i = 0; i < 64; i++)
        {
            PatternOptions.Add(new PatternOption { Index = i, Display = i.ToString() });
            if (!_indexToName.ContainsKey(i)) _indexToName.Add(i, i.ToString());
        }
    }

    // 3) 先依 JSON 建好 TotalOptions（但先不動 Total）
    int def = 22, min = TOTAL_MIN, max = TOTAL_MAX;
    var totalObj = (_cfg != null) ? _cfg["total"] as JObject : null;
    if (totalObj != null)
    {
        def = (int?)totalObj["default"] ?? def;
        min = (int?)totalObj["min"] ?? min;
        max = (int?)totalObj["max"] ?? max;
    }
    BuildTotalOptions(min, max);

    // 4) 若之前按過 Set（_preferHw=true）→ 直接回讀硬體，完全跳過 JSON 重設！
    if (_preferHw)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] LoadFrom(): preferHw=true → skip JSON defaults, refresh HW");
        TryRefreshFromHardware();
        return;
    }

    // 5) 第一次進來（或 Reset 後）：用 JSON default 初始化
    System.Diagnostics.Debug.WriteLine("[AutoRun] LoadFrom(): preferHw=false → use JSON defaults");

    // 設定 Total
    Total = Clamp(def, min, max);     // setter 會自動 ResizeOrders()

    // 初始化 Orders 選項（先清空再建 N 筆）
    Orders.Clear();
    for (int i = 0; i < Total; i++)
        Orders.Add(new OrderSlot { DisplayNo = i + 1, SelectedIndex = 0 });

    // 若 JSON 有 orders.defaults → 套用；否則用前 N 個遞增
    var ordersObj = (_cfg != null) ? _cfg["orders"] as JObject : null;
    var defaults  = (ordersObj != null) ? ordersObj["defaults"] as JArray : null;
    if (defaults != null && defaults.Count > 0)
    {
        for (int i = 0; i < Total && i < defaults.Count; i++)
            Orders[i].SelectedIndex = (int?)defaults[i] ?? 0;
    }
    else
    {
        for (int i = 0; i < Total; i++)
            Orders[i].SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
    }
}