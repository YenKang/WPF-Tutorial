public void LoadFrom(JObject autoRunControl)
{
    if (autoRunControl == null) return;

    _isHydrating = true;
    try
    {
        // 0) 標題
        Title = (string)autoRunControl["title"] ?? "Auto Run";

        // 1) PatternOptions (ItemsSource for Orders)
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

        // 2) TotalOptions + 目標暫存器
        var totalObj = autoRunControl["total"] as JObject;
        int min = totalObj?["min"]?.Value<int>() ?? 1;
        int max = totalObj?["max"]?.Value<int>() ?? 22;
        int def = totalObj?["default"]?.Value<int>() ?? 22;
        _totalTarget = (string)totalObj?["target"] ?? "BIST_PT_TOTAL";

        TotalOptions.Clear();
        for (int i = min; i <= max; i++) TotalOptions.Add(i);

        // ORD0..ORD21 目標（若 JSON 沒給，先用 22 筆預設）
        var ordersObj = autoRunControl["orders"] as JObject;
        _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? InitDefaultOrderTargets();

        // 3) 先決定 UI 來源：Cache 優先，其次 JSON 預設   <<< 這一段就是原本 L142 要搬到這裡
        if (AutoRunCache.HasMemory)
        {
            // (A) 用 Cache 回填 UI（避免紅框：ItemsSource 已經建好）
            Total = Clamp(AutoRunCache.Total, min, max);
            ResizeOrders(Total);

            var arr = AutoRunCache.OrderIndices ?? Array.Empty<int>();
            for (int i = 0; i < Orders.Count && i < arr.Length; i++)
                Orders[i].SelectedIndex = CoercePatternIndex(arr[i]);

            // FCNT1（若你有快取）
            UnpackFcnt1ToUI(AutoRunCache.Fcnt1);
        }
        else
        {
            // (B) 用 JSON default 回填 UI
            Total = Clamp(def, min, max);
            ResizeOrders(Total);
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;
        }

        // 4) 解析 FCNT/trigger 的 mapping（供 ApplyWrite 使用；不要覆寫 UI）
        var f1 = autoRunControl["fcnt1"] as JObject;
        if (f1 != null)
        {
            _fc1Default = ParseHex((string)f1["default"], 0x03C);
            var tgt = f1["targets"] as JObject;
            if (tgt != null)
            {
                _fc1LowTarget  = (string)tgt["low"];
                _fc1HighTarget = (string)tgt["high"];
                _fc1Mask       = ParseHexByte((string)tgt["mask"], 0x03);
                _fc1Shift      = (int?)tgt["shift"] ?? 8;
            }
            if (!AutoRunCache.HasMemory) UnpackFcnt1ToUI(_fc1Default);
        }

        var trig = autoRunControl["trigger"] as JObject;
        _triggerTarget = (string)trig?["target"] ?? "BIST_PT";
        _triggerValue  = ParseHexByte((string)trig?["value"], 0x3F);
    }
    finally { _isHydrating = false; }
}