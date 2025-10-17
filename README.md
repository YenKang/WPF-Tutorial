private static int ResolveSrc(string src, int d2, int d1, int d0, byte low, byte high)
{
    if (string.IsNullOrEmpty(src)) return 0;
    switch (src)
    {
        case "D2": return d2;
        case "D1": return d1;
        case "D0": return d0;
        case "LowByte":  return low;
        case "HighByte": return high;
        default:
            int v;
            return TryParseInt(src, out v) ? v : 0; // 也允許常數
    }
}

private static bool TryParseInt(string s, out int v)
{
    v = 0;
    if (string.IsNullOrWhiteSpace(s)) return false;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
        return int.TryParse(s.Substring(2), System.Globalization.NumberStyles.HexNumber, null, out v);
    return int.TryParse(s, out v);
}

private static int ParseInt(string s)
{
    int v; return TryParseInt(s, out v) ? v : 0;
}