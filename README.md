public void RefreshFromHardware()
{
    if (!_preferHw) return;

    try
    {
        int hwRaw   = RegMap.Read8(REG_TOTAL) & TOTAL_MASK; // 0..21
        int uiTotal = Clamp(hwRaw + 1, TOTAL_MIN, TOTAL_MAX);

        // 確保 TotalOptions 含有 uiTotal（避免 SelectedItem 空白）
        if (TotalOptions.Count == 0 || uiTotal < TotalOptions[0] || uiTotal > TotalOptions[TotalOptions.Count - 1])
        {
            BuildTotalOptions(TOTAL_MIN, TOTAL_MAX);
        }

        Total = uiTotal; // 會 ResizeOrders()

        for (int i = 0; i < Total; i++)
        {
            int v = RegMap.Read8(REG_ORD(i)) & ORD_MASK;
            Orders[i].SelectedIndex = v;
        }

        System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL UI={uiTotal} (raw={hwRaw})");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware failed: " + ex.Message);
    }
}

＝＝＝＝＝
private void Apply()
{
    try
    {
        int uiTotal = Clamp(Total, TOTAL_MIN, TOTAL_MAX);
        byte hwTotal = (byte)((uiTotal - 1) & TOTAL_MASK);
        RegMap.Write8(REG_TOTAL, hwTotal);

        for (int i = 0; i < uiTotal; i++)
        {
            int patternIndex = Orders[i].SelectedIndex & ORD_MASK;
            RegMap.Write8(REG_ORD(i), (byte)patternIndex);
        }
        for (int i = uiTotal; i < TOTAL_MAX; i++)
            RegMap.Write8(REG_ORD(i), 0x00);

        _preferHw = true;          // ← 切回頁面走硬體
        TryRefreshFromHardware();  // ← 寫完立即同步一次（保險）
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
    }
}