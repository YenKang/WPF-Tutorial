// 將 UI 的 D2/D1/D0 打包成 10-bit
private int PackFcnt2FromUI()
{
    int d2 = Clamp(FCNT2_D2, 0, 3);
    int d1 = Clamp(FCNT2_D1, 0, 15);
    int d0 = Clamp(FCNT2_D0, 0, 15);
    return (d2 << 8) | (d1 << 4) | d0;
}

// 將 10-bit 展開到 UI 的 D2/D1/D0
private void UnpackFcnt2ToUI(int value10)
{
    FCNT2_D2 = (value10 >> 8) & 0x03;   // [9:8]
    FCNT2_D1 = (value10 >> 4) & 0x0F;   // [7:4]
    FCNT2_D0 = (value10 >> 0) & 0x0F;   // [3:0]
}