private void Apply()
{
    if (_cfg == null) return;

    // 邊界保護
    HRes = Clamp(HRes, HResMin, HResMax);
    VRes = Clamp(VRes, VResMin, VResMax);

    var m = M <= 0 ? 1 : M;
    var n = N <= 0 ? 1 : N;

    // 基本寬度/高度（整除部分）
    int th = HRes / m;
    int tv = VRes / n;

    // “+1 分配數”＝餘數（有幾個格子是 (basic + 1)）
    int hPlus = HRes - th * m;   // 0..(m-1)
    int vPlus = VRes - tv * n;   // 0..(n-1)

    // ---- H toggle width (13 bits = 8 L + 5 H) ----
    RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", (byte)(th & 0xFF));
    RegMap.Rmw8 ("BIST_CHESSBOARD_H_TOGGLE_W_H", 0x1F, (byte)((th >> 8) & 0x1F));
    // +1 分配數（有幾個格子是 (th+1)）
    RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", (byte)(hPlus & 0xFF));

    // ---- V toggle width (12 bits = 8 L + 4 H) ----
    RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", (byte)(tv & 0xFF));
    RegMap.Rmw8 ("BIST_CHESSBOARD_V_TOGGLE_W_H", 0x0F, (byte)((tv >> 8) & 0x0F));
    // +1 分配數
    RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", (byte)(vPlus & 0xFF));

    // （可選）同步解析度暫存器（若你也維護 0280~0283）
    RegMap.Write8("HRES_Low8",  (byte)(HRes & 0xFF));
    RegMap.Rmw8 ("HRES_High5",  0x1F, (byte)((HRes >> 8) & 0x1F));
    RegMap.Write8("VRES_Low8",  (byte)(VRes & 0xFF));
    RegMap.Rmw8 ("VRES_High4",  0x0F, (byte)((VRes >> 8) & 0x0F));

    // 之後切圖改用硬體真值
    _preferHw = true;

    System.Diagnostics.Debug.WriteLine(
        $"[Chessboard.Apply] HRes={HRes} => th={th}, hPlus={hPlus};  VRes={VRes} => tv={tv}, vPlus={vPlus}; M={m}, N={n}");
}



===

public void RefreshFromRegister()
{
    var m = M <= 0 ? 1 : M;
    var n = N <= 0 ? 1 : N;

    // H: 13 bits + plus-count
    int hL    = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_L");
    int hH    = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H") & 0x1F;
    int th    = (hH << 8) | hL;                  // base width
    int hPlus = RegMap.Read8("BIST_CHESSBOARD_H_PLUS1_BLKNUM") & 0xFF; // +1 分配數

    // V: 12 bits + plus-count
    int vL    = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_L");
    int vH    = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H") & 0x0F;
    int tv    = (vH << 8) | vL;                  // base height
    int vPlus = RegMap.Read8("BIST_CHESSBOARD_V_PLUS1_BLKNUM") & 0xFF; // +1 分配數

    // 還原出解析度：總像素 = base * splits + plusCount
    HRes = Clamp(th * m + hPlus, HResMin, HResMax);
    VRes = Clamp(tv * n + vPlus, VResMin, VResMax);

    System.Diagnostics.Debug.WriteLine(
        $"[Chessboard.Refresh] th={th}, hPlus={hPlus} => HRes={HRes};  tv={tv}, vPlus={vPlus} => VRes={VRes};  M={m}, N={n}");
}