/// <summary>
/// 從 JSON 的 chessH / chessV 節點回讀 toggle 與 plus1 值。
/// 會依據 writes 裡的 target / mask / shift 自動讀取寄存器。
/// </summary>
private void ReadToggleFromJson(JObject chessNode, out int toggle, out int plus1)
{
    toggle = 0;
    plus1 = 0;
    if (chessNode?["writes"] is not JArray writes)
        return;

    string lowTarget = null;
    string highTarget = null;
    string plus1Target = null;
    byte highMask = 0;
    int highShift = 0;

    foreach (JObject w in writes.Cast<JObject>())
    {
        string mode = ((string)w["mode"] ?? "").Trim();
        string target = (string)w["target"];
        string src = ((string)w["src"] ?? "").Trim();
        if (string.IsNullOrWhiteSpace(target))
            continue;

        // 低位：write8 + src=lowByte
        if (mode.Equals("write8", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("lowByte", StringComparison.OrdinalIgnoreCase) &&
            lowTarget == null)
        {
            lowTarget = target;
            continue;
        }

        // 高位：rmw + src=highByte（取 mask / shift）
        if (mode.Equals("rmw", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("highByte", StringComparison.OrdinalIgnoreCase) &&
            highTarget == null)
        {
            highTarget = target;
            highMask = ParseHexByte((string)w["mask"], 0xFF);
            highShift = (int?)w["shift"] ?? 0;
            continue;
        }

        // plus1：write8 + src=lowByte，target 名稱通常帶 PLUS1_BLKNUM
        if (mode.Equals("write8", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("lowByte", StringComparison.OrdinalIgnoreCase) &&
            target.EndsWith("PLUS1_BLKNUM", StringComparison.OrdinalIgnoreCase))
        {
            plus1Target = target;
        }
    }

    // --- 開始實際讀寄存器 ---
    if (lowTarget == null || highTarget == null)
        return;

    int lowVal = RegMap.Read8(lowTarget);
    int highVal = RegMap.Read8(highTarget);
    int highBits = (highVal & highMask) >> highShift;
    toggle = (highBits << 8) | (lowVal & 0xFF);

    if (!string.IsNullOrEmpty(plus1Target))
        plus1 = RegMap.Read8(plus1Target) & 0x01;
}

/// <summary>
/// 將字串（如 "0x1F" 或 "31"）轉成 byte；若失敗回傳預設值。
/// </summary>
private static byte ParseHexByte(string s, byte fallback)
{
    if (string.IsNullOrWhiteSpace(s))
        return fallback;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
        s = s[2..];
    return byte.TryParse(s,
        System.Globalization.NumberStyles.HexNumber,
        System.Globalization.CultureInfo.InvariantCulture,
        out var b)
        ? b
        : fallback;
}