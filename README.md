public void LoadFrom(JObject autoRunControl)
{
    if (autoRunControl == null) return;

    _isHydrating = true;
    try
    {
        // 1️⃣ 標題
        Title = (string)autoRunControl["title"] ?? "Auto Run";

        // 2️⃣ PatternOptions（避免紅框）
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

        // 3️⃣ TotalOptions + 暫存器
        var totalObj = autoRunControl["total"] as JObject;
        int min = totalObj?["min"]?.Value<int>() ?? 1;
        int max = totalObj?["max"]?.Value<int>() ?? 22;
        int def = totalObj?["default"]?.Value<int>() ?? 5;
        _totalTarget = (string)totalObj?["target"] ?? "BIST_PT_TOTAL";

        if (min < 1) min = 1;
        if (max < min) max = min;

        TotalOptions.Clear();
        for (int i = min; i <= max; i++) TotalOptions.Add(i);

        // 4️⃣ Orders 暫存器清單
        var ordersObj = autoRunControl["orders"] as JObject;
        _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? InitDefaultOrderTargets();

        // 5️⃣ 檢查 Cache（PatternOptions 與 TotalOptions 都準備好）
        if (AutoRunCache.HasMemory)
        {
            // (A) 使用快取
            Total = Clamp(AutoRunCache.Total, min, max);
            ResizeOrders(Total);

            var arr = AutoRunCache.OrderIndices ?? Array.Empty<int>();
            for (int i = 0; i < Orders.Count && i < arr.Length; i++)
                Orders[i].SelectedIndex = CoercePatternIndex(arr[i]);

            // 回填 FCNT1
            UnpackFcnt1ToUI(AutoRunCache.Fcnt1);
        }
        else
        {
            // (B) 首次載入：用 JSON default
            Total = Clamp(def, min, max);
            ResizeOrders(Total);
            for (int i = 0; i < Orders.Count; i++)
                Orders[i].SelectedIndex = (i < PatternOptions.Count) ? PatternOptions[i].Index : 0;

            // 解析 FCNT1
            if (autoRunControl["fcnt1"] is JObject f1)
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
                UnpackFcnt1ToUI(_fc1Default);
            }
        }
    }
    finally
    {
        _isHydrating = false;
    }
}