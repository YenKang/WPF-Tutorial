private void ApplyWrite()
{
    Debug.WriteLine("===== [AutoRun] ApplyWrite() start =====");

    if (RegControl == null)
    {
        Debug.WriteLine("[AutoRun] RegControl == null");
        return;
    }

    Debug.WriteLine($"[AutoRun] ApplyWrite() with Total={Total}");

    // (1) TOTAL 仍採用 (Total-1) 規格
    if (!string.IsNullOrEmpty(_totalRegister))
    {
        int t = Total - 1;
        if (t < 0) t = 0;

        // ⭐ 完全 raw，不做 &0x1F
        uint totalRaw = (uint)t;

        Debug.WriteLine(
            $"[AutoRun] Write TOTAL: Reg={_totalRegister}, RawValue(dec)={totalRaw}, Hex=0x{totalRaw:X2}"
        );

        RegControl.SetRegisterByName(_totalRegister, totalRaw);
    }

    // (2) ORD registers — ⭐ 完全 raw 不 clamp、不 mask
    int limit = Math.Min(Total, Orders.Count);

    for (int i = 0; i < limit; i++)
    {
        var slot = Orders[i];

        int raw = slot.SelectedPatternNumber;   // ⭐ 完全 raw

        uint finalValue = (uint)raw;            // ⭐ 無位元遮罩

        Debug.WriteLine(
            $"[AutoRun] ORD[{i}] Reg={slot.RegName}, RawNum={raw}, WriteRaw=0x{finalValue:X2}"
        );

        // ⭐ 完全 raw 寫入
        RegControl.SetRegisterByName(slot.RegName, finalValue);
    }

    Debug.WriteLine("===== [AutoRun] ApplyWrite() end =====");
}
