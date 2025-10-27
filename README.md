// === 新增：切到 GrayLevel 時，從暫存器回讀目前 10-bit，更新三個 ComboBox 的選中值 ===
public void RefreshFromRegister()
{
    try
    {
        // 1) 讀低/高位暫存器（NT51365）
        int low  = RegDisplayControl.ReadRegData(0, (int)NT51365_RegAddr.BIST_PT_LEVEL_L);
        int high = RegDisplayControl.ReadRegData(0, (int)NT51365_RegAddr.BIST_PT_LEVEL_H);

        // 2) 合成 10-bit：高 2bits + 低 8bits
        int v10 = ((high & 0x03) << 8) | (low & 0xFF);

        // 3) 拆回 D2/D1/D0（只改 Selected 值，Options 仍由 JSON 來）
        D2 = (v10 >> 8) & 0x03;   // 0..3
        D1 = (v10 >> 4) & 0x0F;   // 0..15
        D0 = (v10 >> 0) & 0x0F;   // 0..15

        System.Diagnostics.Debug.WriteLine($"[GrayLevel] Refresh OK: {v10:X3}  (D2={D2},D1={D1},D0={D0})");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[GrayLevel] RefreshFromRegister failed: " + ex.Message);
    }
}