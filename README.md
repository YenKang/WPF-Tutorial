public void RefreshFromRegister()
{
    try
    {
        int reg = RegMap.Read8("BIST_GrayColor_VH_Reverse");
        R = (reg & 0x01) != 0 ? 1 : 0;
        G = (reg & 0x02) != 0 ? 1 : 0;
        B = (reg & 0x04) != 0 ? 1 : 0;

        System.Diagnostics.Debug.WriteLine(
            $"[GrayColor] Read BIST_GrayColor_VH_Reverse=0x{reg:X2} => R={R}, G={G}, B={B}");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[GrayColor] RefreshFromRegister failed: " + ex.Message);
    }
}