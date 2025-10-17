private void ExecuteWrite()
{
    // 先把三個值組成 10-bit Gray，並算出 Low/High
    int d2 = (D2 ?? 0) & 0x3;
    int d1 = (D1 ?? 0) & 0xF;
    int d0 = (D0 ?? 0) & 0xF;
    int gray10 = (d2 << 8) | (d1 << 4) | d0;
    byte low  = (byte)(gray10 & 0xFF);
    byte high = (byte)((gray10 >> 8) & 0xFF);

    // 沒有 JSON（或沒 writes）就走既有「寫死」邏輯，避免壞掉
    var writes = _cfg?["writes"] as JArray;
    if (writes == null || writes.Count == 0)
    {
        LegacyWrite(d2, d1, d0, low);
        return;
    }

    // 依 JSON 的每一筆 write 執行
    foreach (JObject w in writes)
    {
        string mode   = (string)w["mode"]   ?? "write8";
        string target = (string)w["target"];
        if (string.IsNullOrEmpty(target)) continue;

        int srcVal = ResolveSrc((string)w["src"], d2, d1, d0, low, high);

        if (mode == "write8")
        {
            RegMap.Write8(target, (byte)(srcVal & 0xFF));
        }
        else if (mode == "rmw")
        {
            int mask  = ParseInt((string)w["mask"]);   // 支援 "0x0C" / "12"
            int shift = (int?)w["shift"] ?? 0;

            byte cur  = RegMap.Read8(target);
            byte next = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
            RegMap.Write8(target, next);
        }
        // 之後要擴充 write16/rmw16... 再加分支
    }
}