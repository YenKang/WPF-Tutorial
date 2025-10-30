public void Apply()
{
    if (Total <= 0) return;

    // --- Step 1: 寫 TOTAL (raw = Total - 1) ---
    byte totalRaw = (byte)((Total - 1) & 0x1F);
    // ⚠️ 改用 Write8，避免上次 RMW 殘留其他 bit 影響硬體判讀
    RegMap.Write8("BIST_PT_TOTAL", totalRaw);
    System.Diagnostics.Debug.WriteLine($"[AutoRun] Write TOTAL raw=0x{totalRaw:X2} (Total={Total})");

    // --- Step 2: 寫每張圖的 ORD ---
    for (int i = 0; i < Total; i++)
    {
        string reg = $"BIST_PT_ORD{i}";
        byte idxRaw = (byte)(Orders[i].SelectedIndex & 0x3F);
        RegMap.Write8(reg, idxRaw);
        System.Diagnostics.Debug.WriteLine($"[AutoRun] Write ORD{i:00} = idx={idxRaw} ({LookupName(idxRaw)})");
    }

    // （可選）把未使用的 ORD 清成 0，避免某些 IC 會誤跑剩餘槽位
    for (int i = Total; i < 22; i++)
    {
        string reg = $"BIST_PT_ORD{i}";
        RegMap.Write8(reg, 0x00);
    }

    _preferHw = true;    // 之後切回此頁，優先讀硬體
    RefreshFromHardware();
}

public void RefreshFromHardware()
{
    try
    {
        // --- 讀 TOTAL ---
        byte totalRaw = RegMap.Read8("BIST_PT_TOTAL");
        int total = (totalRaw & 0x1F) + 1;
        System.Diagnostics.Debug.WriteLine($"[AutoRun] Read TOTAL: raw=0x{totalRaw:X2} -> UI={total}");
        Total = total;  // 這會自動 ResizeOrders(total)

        // --- 讀每張圖 ---
        for (int i = 0; i < total; i++)
        {
            string reg = $"BIST_PT_ORD{i}";
            byte idxRaw = RegMap.Read8(reg);
            int idx = idxRaw & 0x3F;
            EnsureOrderSlot(i).SelectedIndex = idx;
            System.Diagnostics.Debug.WriteLine($"[AutoRun] Read ORD{i:00}: 0x{idxRaw:X2} -> idx={idx} ({LookupName(idx)})");
        }
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] RefreshFromHardware error: " + ex.Message);
    }
}

// 方便除錯：用 options 對照名字
private string LookupName(int idx)
{
    foreach (var p in PatternOptions)
        if (p.Index == idx) return p.Display ?? idx.ToString();
    return idx.ToString();
}