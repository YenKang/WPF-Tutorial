if (IsGrayLevelVisible && p?.GrayLevelControl != null)
{
    string name = p.Name?.ToUpperInvariant() ?? "";

    switch (true)
    {
        case true when name.Contains("PT"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Pattern; break;

        case true when name.Contains("FLK"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Flicker; break;

        case true when name.Contains("GRAY"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Gray; break;

        case true when name.Contains("CROSS"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Crosstalk; break;

        case true when name.Contains("PIXEL"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Pixel; break;

        case true when name.Contains("LINE"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Line; break;

        case true when name.Contains("CURSOR_BG"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.CursorBg; break;

        case true when name.Contains("CURSOR"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Cursor; break;

        case true when name.Contains("CHESS"):
            GrayLevelVM.CurrentRegSet = GrayLevelType.Chessboard; break;

        default:
            GrayLevelVM.CurrentRegSet = GrayLevelType.Pattern; break;
    }

    GrayLevelVM.LoadFrom(p.GrayLevelControl);
    GrayLevelVM.RefreshFromRegister();
}