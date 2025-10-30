private void WriteToRegisters()
{
    int th = HRes / M;
    int tv = VRes / N;

    // --- 寫入 Toggle Width (H/V) ---
    RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", (byte)(th & 0xFF));
    RegMap.Rmw8("BIST_CHESSBOARD_H_TOGGLE_W_H", 0x1F, (byte)((th >> 8) & 0x1F));

    RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", (byte)(tv & 0xFF));
    RegMap.Rmw8("BIST_CHESSBOARD_V_TOGGLE_W_H", 0x0F, (byte)((tv >> 8) & 0x0F));

    // --- 寫入解析度 (H/V) ---
    RegMap.Write8("HRES_Low8", (byte)(HRes & 0xFF));
    RegMap.Rmw8("HRES_High5", 0x1F, (byte)((HRes >> 8) & 0x1F));

    RegMap.Write8("VRES_Low8", (byte)(VRes & 0xFF));
    RegMap.Rmw8("VRES_High4", 0x0F, (byte)((VRes >> 8) & 0x0F));

    System.Diagnostics.Debug.WriteLine(
        $"[Chessboard] Write OK: HRes={HRes}, VRes={VRes}, M={M}, N={N}, th={th}, tv={tv}");
}