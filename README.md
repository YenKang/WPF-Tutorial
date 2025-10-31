public void RefreshFromHardware()
{
    try
    {
        // 1) 讀 TOTAL（0 表示 1 張）
        byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
        int total = ((totalRaw & 0x1F) + 1);
        Total = Clamp(total, 1, 22); // 內部會呼叫 ResizeOrders()

        // 2) 先把硬體的 ORD 值讀進暫存陣列
        var idxBuf = new int[Total];
        for (int i = 0; i < Total; i++)
        {
            string reg = $"BIST_PT_ORD{i}";
            byte v = RegMap.Read8(reg);
            idxBuf[i] = v & 0x3F;
        }

        // 3) 批次灌值：暫停刷新 + 關閉 hydration
        var view = CollectionViewSource.GetDefaultView(Orders);
        using (view?.DeferRefresh())
        {
            _isHydrating = true;
            for (int i = 0; i < idxBuf.Length; i++)
                EnsureOrderSlot(i).SelectedIndex = idxBuf[i];
            _isHydrating = false;
        }

        System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware OK.");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
    }
}

＝＝＝＝＝
public void LoadFrom(JObject autoRunControl, bool preferHw = false)
{
    if (autoRunControl == null) return;

    _cfg = autoRunControl;
    _preferHw = preferHw;

    Title = (string)autoRunControl["title"] ?? "Auto Run";

    // 先建 PatternOptions（一次就緒）
    PatternOptions.Clear();
    var optArr = autoRunControl["options"] as JArray;
    if (optArr != null)
    {
        foreach (var t in optArr.OfType<JObject>())
        {
            int idx  = t["index"]?.Value<int>() ?? 0;
            string name = (string)t["name"] ?? idx.ToString();
            PatternOptions.Add(new PatternOption { Index = idx, Display = name });
        }
    }

    // 先建 TotalOptions（一次就緒）
    TotalOptions.Clear();
    var totalObj = autoRunControl["total"] as JObject;
    int min = totalObj?["min"]?.Value<int>() ?? 1;
    int max = totalObj?["max"]?.Value<int>() ?? 22;
    int def = totalObj?["default"]?.Value<int>() ?? 22;
    for (int i = min; i <= max; i++) TotalOptions.Add(i);

    // 這時才設 Total（避免紅框）
    Total = Clamp(def, min, max);

    // 先依 JSON default 灌初值（畫面立即有內容，不空白）
    using (CollectionViewSource.GetDefaultView(Orders)?.DeferRefresh())
    {
        _isHydrating = true;
        ResizeOrders(Total);
        for (int i = 0; i < Orders.Count; i++)
            Orders[i].SelectedIndex = i < PatternOptions.Count ? PatternOptions[i].Index : 0;
        _isHydrating = false;
    }

    // 若 preferHw，**最後**一次性覆寫為硬體真值
    if (_preferHw) RefreshFromHardware();
}