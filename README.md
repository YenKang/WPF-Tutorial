/// <summary>
/// 讀取棋盤格暫存器（依 JSON 的 writes 找 target/mask/shift），
/// 還原當前的 HRes/VRes（用最小符合法），並更新 UI。
/// </summary>
public void RefreshFromRegister()
{
    try
    {
        // === H 軸：從 json 的 chessH 解析 ===
        ReadToggleFromJson(_jChessH, out int th, out int hPlus1);
        // === V 軸：從 json 的 chessV 解析 ===
        ReadToggleFromJson(_jChessV, out int tv, out int vPlus1);

        // 依目前 M/N 反推 Resolution（最小符合：整除=th*M；有 plus1 → +1）
        if (M > 0)
        {
            int backH = th * M + (hPlus1 != 0 ? 1 : 0);
            HRes = Clamp(backH, HResMin, HResMax);
        }
        if (N > 0)
        {
            int backV = tv * N + (vPlus1 != 0 ? 1 : 0);
            VRes = Clamp(backV, VResMin, VResMax);
        }

        System.Diagnostics.Debug.WriteLine(
            $"[Chessboard] th={th}, +{hPlus1}; tv={tv}, +{vPlus1} => HRes={HRes}, VRes={VRes}, M={M}, N={N}");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[Chessboard] RefreshFromRegister failed: " + ex.Message);
    }
}