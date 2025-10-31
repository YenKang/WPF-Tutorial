public void LoadFrom(JObject autoRunControl)
{
    if (autoRunControl == null) return;

    // (0) 標題
    Title = (string)autoRunControl["title"] ?? "Auto Run";

    // (0.5) 先把 Cache 拿起來 → 存成 pending，之後再套用
    _hasPendingCache = AutoRunCache.HasMemory;
    if (_hasPendingCache)
    {
        _pendingTotal = AutoRunCache.Total;
        _pendingOrderIdx = AutoRunCache.OrderIndices ?? new int[0];
        _pendingFcnt1 = AutoRunCache.Fcnt1;
        _pendingFcnt2 = AutoRunCache.Fcnt2;
    }
    else
    {
        _pendingTotal = 0;
        _pendingOrderIdx = new int[0];
        _pendingFcnt1 = _pendingFcnt2 = 0;
    }

    // (1) Pattern 選單
    PatternOptions.Clear();
    var optArr = autoRunControl["options"] as JArray;
    if (optArr != null)
    {
        foreach (var t in optArr.OfType<JObject>())
        {
            int idx = t["index"]?.Value<int>() ?? 0;
            string name = (string)t["name"] ?? idx.ToString();
            PatternOptions.Add(new PatternOption { Index = idx, Display = name });
        }
    }

    // (2) Total 設定(含選單)
    var totalObj = autoRunControl["total"] as JObject;
    int min = totalObj?["min"]?.Value<int>() ?? 1;
    int max = totalObj?["max"]?.Value<int>() ?? 22;
    int def = totalObj?["default"]?.Value<int>() ?? 22;
    _totalTarget   = (string)totalObj?["target"] ?? "BIST_PT_TOTAL";

    TotalOptions.Clear();
    for (int i = min; i <= max; i++) TotalOptions.Add(i);

    // 若有 pending，就用 cache 的 total；否則用 JSON default
    int initTotal = _hasPendingCache ? Clamp(_pendingTotal, min, max) : Clamp(def, min, max);
    Total = initTotal;                 // 會呼叫 ResizeOrders(initTotal)
    if (Orders.Count == 0) ResizeOrders(initTotal);

    // (3) Orders 目標暫存器
    var ordersObj = autoRunControl["orders"] as JObject;
    _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? new string[0];

    // (4) FCNT1
    if (autoRunControl["fcnt1"] is JObject f1)
    {
        _fc1Default = ParseHex((string)f1["default"], 0x03C);
        if (f1["targets"] is JObject tgt)
        {
            _fc1LowTarget  = (string)tgt["low"];
            _fc1HighTarget = (string)tgt["high"];
        }
    }

    // (5) FCNT2
    if (autoRunControl["fcnt2"] is JObject f2)
    {
        _fc2Default = ParseHex((string)f2["default"], 0x01E);
        if (f2["targets"] is JObject tgt)
        {
            _fc2LowTarget  = (string)tgt["low"];
            _fc2HighTarget = (string)tgt["high"];
        }
    }

    // (6) Trigger
    if (autoRunControl["trigger"] is JObject trig)
    {
        _triggerTarget = (string)trig["target"];
        _triggerValue  = ParseHexByte((string)trig["value"], 0x3F);
    }

    // (7) 現在再把 pending cache 套回 UI（此時 ItemsSource 都 ready）
    if (_hasPendingCache)
    {
        // Orders 選擇
        for (int i = 0; i < Math.Min(Orders.Count, _pendingOrderIdx.Length); i++)
            Orders[i].SelectedIndex = _pendingOrderIdx[i];

        // FCNT
        SetFcntDigitsFromValue(_pendingFcnt1, true);
        SetFcntDigitsFromValue(_pendingFcnt2, false);
    }
    else
    {
        // 沒 cache → 用 JSON 預設 FCNT
        SetFcntDigitsFromValue(_fc1Default, true);
        SetFcntDigitsFromValue(_fc2Default, false);
    }

    // 套用一次後就清掉 pending 標記
    _hasPendingCache = false;
}