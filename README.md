public void LoadFrom(JObject autoRunControl)
{
    if (autoRunControl == null) return;

    // 1) 標題
    Title = (string)autoRunControl["title"] ?? "Auto Run";

    // 2) 若有 Cache -> 直接還原 UI（最快讓 UI 有值，避免紅框）
    if (AutoRunCache.HasMemory)
    {
        RestoreFromCache();  // 只動 UI，不讀硬體
        // 仍然要解析 JSON（在下面步驟）以取得 registers/mapping 供 Apply 使用
    }

    // 3) 解析 PatternOptions（index/name）
    PatternOptions.Clear();
    var optArr = autoRunControl["options"] as JArray;
    if (optArr != null)
    {
        foreach (var t in optArr.OfType<JObject>())
        {
            int idx  = t["index"] != null ? t["index"].Value<int>() : 0;
            string name = (string)t["name"] ?? idx.ToString();
            PatternOptions.Add(new PatternOption { Index = idx, Display = name });
        }
    }

    // 4) 解析 total 的 min/max/default/target
    var totalObj = autoRunControl["total"] as JObject;
    int min = totalObj?["min"] != null ? totalObj["min"].Value<int>() : 1;
    int max = totalObj?["max"] != null ? totalObj["max"].Value<int>() : 22;
    int def = totalObj?["default"] != null ? totalObj["default"].Value<int>() : 22;
    _totalTarget = (string)totalObj?["target"] ?? "BIST_PT_TOTAL";

    TotalOptions.Clear();
    for (int i = min; i <= max; i++) TotalOptions.Add(i);

    if (!AutoRunCache.HasMemory)
    {
        // 初次：用 JSON default 填 UI
        Total = Clamp(def, min, max);
        ResizeOrders(Total);
        for (int i = 0; i < Orders.Count; i++)
            Orders[i].SelectedIndex = i < PatternOptions.Count ? PatternOptions[i].Index : 0;
    }
    else
    {
        // Cache 已經把 Total / Orders 指好了，這裡只要確保在合法範圍
        Total = Clamp(Total, min, max);
        ResizeOrders(Total);
    }

    // 5) 讀 FCNT1 映射 + 預設
    var f1 = autoRunControl["fcnt1"] as JObject;
    _fc1Default = ParseHex((string)f1?["default"], 0x03C);

    var tgt = f1?["targets"] as JObject;
    if (tgt != null)
    {
        _fc1LowTarget  = (string)tgt["low"];
        _fc1HighTarget = (string)tgt["high"];
        _fc1Mask       = ParseHexByte((string)tgt["mask"], 0x03);
        _fc1Shift      = tgt["shift"] != null ? tgt["shift"].Value<int>() : 8;
    }

    // 6) 設定 FCNT1 UI：Cache 優先，否則用 JSON default
    int f1Val = AutoRunCache.HasMemory ? AutoRunCache.Fcnt1 : _fc1Default;
    UnpackFcnt1ToUI(f1Val);   // 轉為 D2/D1/D0，並自動 clamp
}