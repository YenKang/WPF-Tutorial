private void Apply()
{
    if (_cfg == null) return;

    // 1️⃣ Clamp 使用者輸入（防止越界）
    HRes = Clamp(HRes, HResMin, HResMax);
    VRes = Clamp(VRes, VResMin, VResMax);

    int m = M, n = N;
    int hRes = HRes, vRes = VRes;

    // 2️⃣ 計算每格寬/高 (Toggle Width) 與 +1旗標
    int th     = (m > 0) ? (hRes / m) : 0;
    int hPlus1 = (m > 0 && (hRes % m) != 0) ? 1 : 0;

    int tv     = (n > 0) ? (vRes / n) : 0;
    int vPlus1 = (n > 0 && (vRes % n) != 0) ? 1 : 0;

    // 限制位寬 (H=13bit, V=12bit)
    th = ClampToBitWidth(th, 8, 5);
    tv = ClampToBitWidth(tv, 8, 4);

    // 拆成低/高位
    byte h_low  = (byte)(th & 0xFF);
    byte h_high = (byte)((th >> 8) & 0x1F);  // [4:0]
    byte v_low  = (byte)(tv & 0xFF);
    byte v_high = (byte)((tv >> 8) & 0x0F);  // [3:0]

    // 3️⃣ 寫入「棋盤格寬度」H方向 (003C~0040)
    RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", h_low);
    RegMap.Rmw8 ("BIST_CHESSBOARD_H_TOGGLE_W_H", 0x1F, h_high);
    RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", (byte)hPlus1);   // ✅ 用 Write8

    // 4️⃣ 寫入「棋盤格高度」V方向 (003E~0041)
    RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", v_low);
    RegMap.Rmw8 ("BIST_CHESSBOARD_V_TOGGLE_W_H", 0x0F, v_high);
    RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", (byte)vPlus1);   // ✅ 用 Write8

    // 5️⃣ 寫入解析度 (0280~0283)
    RegMap.Write8("HRES_Low8",  (byte)(hRes & 0xFF));
    RegMap.Rmw8 ("HRES_High5",  0x1F, (byte)((hRes >> 8) & 0x1F));
    RegMap.Write8("VRES_Low8",  (byte)(vRes & 0xFF));
    RegMap.Rmw8 ("VRES_High4",  0x0F, (byte)((vRes >> 8) & 0x0F));

    // 6️⃣ 啟用硬體真值模式
    _preferHw = true;

    System.Diagnostics.Debug.WriteLine(
        $"[Chessboard] Apply OK: HRes={hRes}, VRes={vRes}, " +
        $"M={m}, N={n}, th={th}(+{hPlus1}), tv={tv}(+{vPlus1})");
}

// 位寬安全裁切
private static int ClampToBitWidth(int value, int lowBits, int highBits)
{
    if (value < 0) return 0;
    int totalBits = lowBits + highBits;
    int max = (1 << totalBits) - 1;
    return (value > max) ? max : value;
}