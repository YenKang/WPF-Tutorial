// 依 datasheet 調整這裡
private static (string reg, byte mask, int shift) GetAxisSpec(string axis)
{
    const string REG = "BIST_GrayColor_VH_Reverse";
    return (axis == "V")
        ? (REG, (byte)0x20, 5)  // V: bit5
        : (REG, (byte)0x10, 4); // H: bit4
}