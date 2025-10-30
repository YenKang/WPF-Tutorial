public void RefreshFromRegister()
{
    try
    {
        // 1) 讀 toggle width + plus-count
        int hL    = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_L");
        int hH    = RegMap.Read8("BIST_CHESSBOARD_H_TOGGLE_W_H") & 0x1F;
        int th    = (hH << 8) | hL; // base width (H)

        int vL    = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_L");
        int vH    = RegMap.Read8("BIST_CHESSBOARD_V_TOGGLE_W_H") & 0x0F;
        int tv    = (vH << 8) | vL; // base height (V)

        int hPlus = RegMap.Read8("BIST_CHESSBOARD_H_PLUS1_BLKNUM") & 0xFF; // +1 分配數
        int vPlus = RegMap.Read8("BIST_CHESSBOARD_V_PLUS1_BLKNUM") & 0xFF;

        // 2) 讀解析度真值 (0280..0283)
        int hResLow  = RegMap.Read8("HRES_Low8");
        int hResHigh = RegMap.Read8("HRES_High5") & 0x1F;
        int vResLow  = RegMap.Read8("VRES_Low8");
        int vResHigh = RegMap.Read8("VRES_High4") & 0x0F;

        int hResReg = ((hResHigh << 8) | hResLow) & 0x1FFF;
        int vResReg = ((vResHigh << 8) | vResLow) & 0x0FFF;

        // 3) 先用“真值解析度”覆蓋 UI
        HRes = Clamp(hResReg, HResMin, HResMax);
        VRes = Clamp(vResReg, VResMin, VResMax);

        // 4) 再由 th/tv 和 plus-count 反推出 M/N（避免被 JSON 預設汙染）
        if (th > 0)
        {
            int mInf = (HRes - hPlus) / th;     // 整除即對；有誤差時會向下取整
            M = Clamp(mInf, MMin, MMax);
        }
        if (tv > 0)
        {
            int nInf = (VRes - vPlus) / tv;
            N = Clamp(nInf, NMin, NMax);
        }

        System.Diagnostics.Debug.WriteLine(
            $"[Chessboard.Refresh] th={th}, hPlus={hPlus} => HRes={HRes}, M={M}; " +
            $"tv={tv}, vPlus={vPlus} => VRes={VRes}, N={N}");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[Chessboard] RefreshFromRegister failed: " + ex.Message);
    }
}