public void RefreshFromRegister()
{
    try
    {
        ReadToggleFromJson(_jChessH, out int th, out int hPlus1);
        ReadToggleFromJson(_jChessV, out int tv, out int vPlus1);

        if (M > 0)
            HRes = Clamp(th * M + (hPlus1 != 0 ? 1 : 0), HResMin, HResMax);
        if (N > 0)
            VRes = Clamp(tv * N + (vPlus1 != 0 ? 1 : 0), VResMin, VResMax);

        System.Diagnostics.Debug.WriteLine(
            $"[Chessboard] RefreshFromRegister OK → th={th}, tv={tv}, HRes={HRes}, VRes={VRes}");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[Chessboard] RefreshFromRegister failed: " + ex.Message);
    }
}