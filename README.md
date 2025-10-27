/// <summary>
/// 從暫存器讀取目前的 Gray Level (10-bit)，更新 D2/D1/D0 三個 UI 綁定值。
/// </summary>
public void RefreshFromRegister()
{
    try
    {
        // --- 讀取寄存器資料 ---
        // BIST_PT_LEVEL : 低 8 bits (位址 0x0030)
        int levelLowByte = RegMap.Read8("BIST_PT_LEVEL");

        // BIST_CK_GY_FK_PT : 只取 bit[1:0] 為 Gray Level 的高 2 bits (位址 0x0034)
        int fkPtRegValue = RegMap.Read8("BIST_CK_GY_FK_PT");
        int levelHighBits = fkPtRegValue & 0x03;

        // --- 合成完整 10-bit Gray Level ---
        int grayLevel10bit = (levelHighBits << 8) | (levelLowByte & 0xFF);

        // --- 拆解成三個 UI Digit ---
        D2 = (grayLevel10bit >> 8) & 0x03;   // bit[9:8]
        D1 = (grayLevel10bit >> 4) & 0x0F;   // bit[7:4]
        D0 =  grayLevel10bit        & 0x0F;  // bit[3:0]

        // --- Debug Log ---
        System.Diagnostics.Debug.WriteLine(
            $"[GrayLevel] Read OK → 0030h(BIST_PT_LEVEL)={levelLowByte:X2}, " +
            $"0034h(BIST_CK_GY_FK_PT)={fkPtRegValue:X2}, Result={grayLevel10bit:X3} " +
            $"(D2={D2}, D1={D1}, D0={D0})"
        );
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"[GrayLevel] RefreshFromRegister failed: {ex.Message}");
    }
}

