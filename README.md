private static int Clamp(int v, int min, int max)
{
    if (v < min) return min;
    if (v > max) return max;
    return v;
}

private static byte ParseHexByte(string s, byte fallback)
{
    if (string.IsNullOrWhiteSpace(s)) return fallback;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
    byte b;
    return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out b) ? b : fallback;
}

private static int ParseHex(string s, int fallback)
{
    if (string.IsNullOrWhiteSpace(s)) return fallback;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
    int v;
    return int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out v) ? v : fallback;
}

private void UnpackFcnt1ToUI(int value10)
{
    value10 &= 0x3FF;
    FCNT1_D2 = (value10 >> 8) & 0x3;  // 2 bits
    FCNT1_D1 = (value10 >> 4) & 0xF;  // 4 bits
    FCNT1_D0 = (value10 >> 0) & 0xF;  // 4 bits
}

private int PackFcnt1FromUI()
{
    var d2 = FCNT1_D2 & 0x3;
    var d1 = FCNT1_D1 & 0xF;
    var d0 = FCNT1_D0 & 0xF;
    return ((d2 << 8) | (d1 << 4) | d0) & 0x3FF;
}