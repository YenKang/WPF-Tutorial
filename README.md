if (IsGrayLevelVisible && p?.GrayLevelControl != null)
{
    // 依你的圖名/索引決定類型（下面僅示例，請換成你實際判斷）
    if      (p.Name.Contains("PT"))        GrayLevelVM.CurrentRegSet = GrayLevelType.Pattern;
    else if (p.Name.Contains("FLK"))       GrayLevelVM.CurrentRegSet = GrayLevelType.Flicker;
    else if (p.Name.Contains("GRAY"))      GrayLevelVM.CurrentRegSet = GrayLevelType.Gray;
    else if (p.Name.Contains("CROSS"))     GrayLevelVM.CurrentRegSet = GrayLevelType.Crosstalk;
    else if (p.Name.Contains("PIXEL"))     GrayLevelVM.CurrentRegSet = GrayLevelType.Pixel;
    else if (p.Name.Contains("LINE"))      GrayLevelVM.CurrentRegSet = GrayLevelType.Line;
    else if (p.Name.Contains("CURSOR_BG")) GrayLevelVM.CurrentRegSet = GrayLevelType.CursorBg;
    else if (p.Name.Contains("CURSOR"))    GrayLevelVM.CurrentRegSet = GrayLevelType.Cursor;
    else if (p.Name.Contains("CHESS"))     GrayLevelVM.CurrentRegSet = GrayLevelType.Chessboard;

    GrayLevelVM.LoadFrom(p.GrayLevelControl); // 照舊：JSON 生 options
    GrayLevelVM.RefreshFromRegister();        // 新增：用目前暫存器覆寫 Selected 值
}