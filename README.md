private static int ParseHex(string s, int fallback)
{
    if (string.IsNullOrWhiteSpace(s)) return fallback;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
    int v;
    return int.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out v) ? v : fallback;
}

private static byte ParseHexByte(string s, byte fallback)
{
    if (string.IsNullOrWhiteSpace(s)) return fallback;
    s = s.Trim();
    if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) s = s.Substring(2);
    byte v;
    return byte.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out v) ? v : fallback;
}


// Step 2.5: 寫入 FCNT1 / FCNT2（若 JSON 提供 mapping）
WriteFcnt(_fc1LowTarget, _fc1HighTarget, _fc1Mask, _fc1Shift,
          (FCNT1_D2 << 8) | (FCNT1_D1 << 4) | FCNT1_D0);

WriteFcnt(_fc2LowTarget, _fc2HighTarget, _fc2Mask, _fc2Shift,
          (FCNT2_D2 << 8) | (FCNT2_D1 << 4) | FCNT2_D0);

===


private static void WriteFcnt(string lowTarget, string highTarget, byte mask, int shift, int value10bit)
{
    if (!string.IsNullOrEmpty(lowTarget))
        RegMap.Write8(lowTarget, (byte)(value10bit & 0xFF));

    if (!string.IsNullOrEmpty(highTarget) && mask != 0)
    {
        // 依 JSON 的 shift/mask 對齊
        byte hi = (byte)((value10bit >> shift) & mask);
        RegMap.Rmw(highTarget, mask, hi);
    }
}