private void UpdateIcDlSel()
{
    // 1. 若勾選 Broadcast → 固定 0x4
    if (IsBroadcastEn)
    {
        IcDlSel = 0x4;   // 100b
        return;
    }

    // 2. 沒勾 Broadcast：依 SelectedIC (ICIndex enum) 決定數值
    int icDlSelValue;

    switch (SelectedIC)
    {
        case ICIndex.Primary:
            icDlSelValue = 0x0;   // 000b
            break;

        case ICIndex.L1:
            icDlSelValue = 0x1;   // 001b
            break;

        case ICIndex.L2:
            icDlSelValue = 0x2;   // 010b
            break;

        case ICIndex.L3:
            icDlSelValue = 0x3;   // 011b
            break;

        default:
            icDlSelValue = 0x0;   // 安全預設
            break;
    }

    IcDlSel = icDlSelValue;
}
