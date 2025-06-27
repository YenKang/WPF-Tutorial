private int ComputeDelta(int current, int previous)
{
    int rawDelta = current - previous;

    // 處理 overflow（0 ~ 255 的旋鈕）
    if (rawDelta > 128)       // 比 128 還大，表示其實是逆轉回去
        rawDelta -= 256;
    else if (rawDelta < -128) // 比 -128 還小，表示其實是順轉一圈
        rawDelta += 256;

    return rawDelta;
}