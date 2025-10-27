private int Read10Bit(string lowReg, string highReg, byte mask, int shift)
{
    int low  = RegMap.Read8(lowReg) & 0xFF;
    int high = RegMap.Read8(highReg);
    int hi2  = (high & mask) >> shift;         // 取出高2bit（位於 high 的 mask 區，右移到 LSB）
    return ((hi2 & 0x03) << 8) | low;          // 合成 10-bit
}

private void ApplyGrayDigits(int gray10)
{
    if (gray10 < 0) gray10 = 0;
    if (gray10 > 0x3FF) gray10 = 0x3FF;
    D2 = (gray10 >> 8) & 0x03;   // [9:8]
    D1 = (gray10 >> 4) & 0x0F;   // [7:4]
    D0 =  gray10        & 0x0F;  // [3:0]
}