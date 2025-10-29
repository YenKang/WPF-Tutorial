private void WriteToRegisters()
{
    int th = HRes / M;
    int hPlus1 = (HRes % M) != 0 ? 1 : 0;

    int tv = VRes / N;
    int vPlus1 = (VRes % N) != 0 ? 1 : 0;

    // === H 軸 ===
    RegMap.Write8("BIST_CHESSBOARD_H_TOGGLE_W_L", (byte)(th & 0xFF));

    // 高 5 bits 在 [4:0] → 不用位移；mask=0x1F，valueShifted= (th >> 8) & 0x1F
    RegMap.Rmw8("BIST_CHESSBOARD_H_TOGGLE_W_H",
                (byte)0x1F,
                (byte)((th >> 8) & 0x1F));

    RegMap.Write8("BIST_CHESSBOARD_H_PLUS1_BLKNUM", (byte)(hPlus1 & 0x01));

    // === V 軸 ===
    RegMap.Write8("BIST_CHESSBOARD_V_TOGGLE_W_L", (byte)(tv & 0xFF));

    // 高 4 bits 在 [3:0] → 不用位移；mask=0x0F，valueShifted= (tv >> 8) & 0x0F
    RegMap.Rmw8("BIST_CHESSBOARD_V_TOGGLE_W_H",
                (byte)0x0F,
                (byte)((tv >> 8) & 0x0F));

    RegMap.Write8("BIST_CHESSBOARD_V_PLUS1_BLKNUM", (byte)(vPlus1 & 0x01));
}