private void Apply()
{
    try
    {
        // (可選) 先停 Auto-Run 觸發，避免寫入中間被硬體讀到
        // RegMap.Write8("BIST_AUTORUN_TRIG", 0x00); // 若你有這顆暫存器

        // 1) 寫入 Total（[4:0]）
        byte totalVal = (byte)(Total & 0x1F);
        RegMap.Write8("BIST_PT_TOTAL", totalVal);

        // 2) 寫入有效的 ORD 槽位（0..Total-1）
        for (int i = 0; i < Orders.Count; i++)
        {
            int idx = Clamp(Orders[i].SelectedIndex, 0, 21);
            RegMap.Write8("BIST_PT_ORD" + i, (byte)(idx & 0x3F)); // 題示：索引用 [5:0]
        }

        // 3) 清掉後面沒用到的 ORD（Total..21），避免殘值被硬體讀到
        for (int i = Orders.Count; i < 22; i++)
        {
            RegMap.Write8("BIST_PT_ORD" + i, 0x00); // 這裡我清成 0=Black，你也可改成最後一張或固定保底
        }

        // 4) (可選) 重新觸發 Auto-Run
        // RegMap.Write8("BIST_AUTORUN_TRIG", 0x01);

        // 5) 之後以硬體真值為主
        _preferHw = true;

        // 6) 讀回核對（方便你現在找問題）
        int hwTotal = RegMap.Read8("BIST_PT_TOTAL") & 0x1F;
        System.Diagnostics.Debug.WriteLine($"[AutoRun] Write OK → Total={Total} (HW={hwTotal})");
        for (int i = 0; i < 22; i++)
        {
            int v = RegMap.Read8("BIST_PT_ORD" + i) & 0x3F;
            System.Diagnostics.Debug.WriteLine($"  ORD{i} = {v}");
        }

        // 7) 立刻用硬體數值刷新 UI（維持你既定行為）
        TryRefreshFromHardware();
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[AutoRun] Apply failed: " + ex.Message);
    }
}

