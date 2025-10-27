public void RefreshFromRegister()
{
    try
    {
        int gray10 = 0;

        switch (CurrentRegSet)
        {
            case GrayLevelType.Pattern:
                gray10 = Read10Bit("BIST_PT_LEVEL", "BIST_CK_GY_FK_PT", 0x03, 0);
                break;
            case GrayLevelType.Flicker:
                gray10 = Read10Bit("BIST_FLK_LEVEL", "BIST_CK_GY_FK_PT", 0x0C, 2);
                break;
            case GrayLevelType.Gray:
                gray10 = Read10Bit("BIST_GRAY_LEVEL", "BIST_CK_GY_FK_PT", 0x30, 4);
                break;
            case GrayLevelType.Crosstalk:
                gray10 = Read10Bit("BIST_CROSSTALK_LEVEL", "BIST_CK_GY_FK_PT", 0xC0, 6);
                break;
            case GrayLevelType.Pixel:
                gray10 = Read10Bit("BIST_PIXEL_LEVEL", "BIST_CB_CL_LL_PL", 0x03, 0);
                break;
            case GrayLevelType.Line:
                gray10 = Read10Bit("BIST_LINE_LEVEL", "BIST_CB_CL_LL_PL", 0x0C, 2);
                break;
            case GrayLevelType.Cursor:
                gray10 = Read10Bit("BIST_CURSOR_LEVEL", "BIST_CB_CL_LL_PL", 0x30, 4);
                break;
            case GrayLevelType.CursorBg:
                gray10 = Read10Bit("BIST_CURSOR_BG_LEVEL", "BIST_CB_CL_LL_PL", 0xC0, 6);
                break;
            case GrayLevelType.Chessboard: // 🆕 新增棋盤圖模式
                gray10 = Read10Bit("BIST_CHESSBOARD_LEVEL", "BIST_CHESSBOARD_HI", 0x03, 0);
                break;
        }

        ApplyGrayDigits(gray10);
        System.Diagnostics.Debug.WriteLine($"[GrayLevel] {CurrentRegSet} => {gray10:X3} (D2={D2}, D1={D1}, D0={D0})");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"[GrayLevel] RefreshFromRegister failed: {ex.Message}");
    }
}