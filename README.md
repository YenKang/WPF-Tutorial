private void ExecuteWrite()
{
    bool ok = false;

    try
    {
        int r = R & 0x1, g = G & 0x1, b = B & 0x1;

        if (!(_cfg?["writes"] is JArray writes) || writes.Count == 0)
        {
            // Fallback：一次 RMW 寫入 0x004F 的 bit[2:0]
            byte cur  = RegMap.Read8("BIST_GrayColor_VH_Reverse");
            byte next = (byte)((cur & ~0x07) | (r << 0) | (g << 1) | (b << 2));
            RegMap.Write8("BIST_GrayColor_VH_Reverse", next);

            ok = true;                    // ← 成功
        }
        else
        {
            foreach (JObject w in writes.Cast<JObject>())
            {
                string mode   = (string)w["mode"] ?? "rmw";
                string target = (string)w["target"]; if (string.IsNullOrEmpty(target)) continue;
                string srcStr = (string)w["src"];
                int   srcVal  = ResolveSrc(srcStr, r, g, b);

                if (mode == "write8")
                {
                    RegMap.Write8(target, (byte)(srcVal & 0xFF));
                    continue;
                }

                int mask  = ParseInt((string)w["mask"]);
                int shift = (int?)w["shift"] ?? 0;

                byte cur  = RegMap.Read8(target);
                byte next = (byte)((cur & ~mask & 0xFF) | (((srcVal << shift) & mask) & 0xFF));
                RegMap.Write8(target, next);
            }

            ok = true;                    // ← 成功
        }
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[GrayColor] Write failed: " + ex.Message);
    }

    if (ok)
    {
        _preferHw = true;                 // ← 只在成功時設一次
        TryRefreshFromRegister();         // 立刻回讀同步 UI
        System.Diagnostics.Debug.WriteLine($"[GrayColor] Write OK: R={R} G={G} B={B}");
    }
}