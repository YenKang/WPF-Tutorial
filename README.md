/// <summary>
/// 從一個 chess 節點（"chessH" 或 "chessV"）回讀 toggle 與 plus1。
/// JSON 範例：
///   write8  target=..._TOGGLE_W_L      src=lowByte
///   rmw     target=..._TOGGLE_W_H mask=0x1F / 0x0F shift=0  src=highByte
///   write8  target=..._PLUS1_BLKNUM    src=lowByte (bit0)
/// </summary>
private void ReadToggleFromJson(JObject chessNode, out int toggle, out int plus1)
{
    toggle = 0; plus1 = 0;
    if (chessNode?["writes"] is not JArray writes) return;

    string lowTarget   = null;
    string highTarget  = null;
    byte   highMask    = 0;
    int    highShift   = 0;
    string plus1Target = null;

    foreach (JObject w in writes.Cast<JObject>())
    {
        string mode   = ((string)w["mode"] ?? "").Trim();
        string target = (string)w["target"];
        string src    = ((string)w["src"] ?? "").Trim();

        if (string.IsNullOrWhiteSpace(target)) continue;

        // 低位：write8 + src=lowByte
        if (mode.Equals("write8", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("lowByte", StringComparison.OrdinalIgnoreCase) &&
            lowTarget == null)
        {
            lowTarget = target;
            continue;
        }

        // 高位：rmw + src=highByte（取 mask/shift）
        if (mode.Equals("rmw", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("highByte", StringComparison.OrdinalIgnoreCase) &&
            highTarget == null)
        {
            highTarget = target;
            highMask   = ParseHexByte((string)w["mask"], 0xFF);
            highShift  = (int?)w["shift"] ?? 0;
            continue;
        }

        // plus1：write8 + src=lowByte（通常是 _PLUS1_BLKNUM）
        if (mode.Equals("write8", StringComparison.OrdinalIgnoreCase) &&
            src.Equals("lowByte", StringComparison.OrdinalIgnoreCase) &&
            target.EndsWith("PLUS1_BLKNUM", StringComparison.OrdinalIgnoreCase))
        {
            plus1Target = target;
        }
    }

    if (lowTarget == null || highTarget == null)
        return; // JSON 不完整就不處理

    int low  = RegMap.Read8(lowTarget);
    int hreg = RegMap.Read8(highTarget);
    int hib  = (hreg & highMask) >> highShift;
    toggle   = (hib << 8) | (low & 0xFF);

    if (!string.IsNullOrEmpty(plus1Target))
        plus1 = RegMap.Read8(plus1Target) & 0x01;
}

private static byte ParseHexByte(string s, byte fb)
{
    if (string.IsNullOrWhiteSpace(s)) return fb;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s[2..];
    return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber,
        System.Globalization.CultureInfo.InvariantCulture, out var b) ? b : fb;
}