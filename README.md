private void ExecuteWrite()
{
    try
    {
        var (reg, mask, shift) = GetAxisSpec(Axis);

        // Rmw8(name, mask, valueShifted) 的第三參數要「先位移好」再丟
        byte valueShifted = (byte)(((Enable ? 1 : 0) << shift) & mask);

        RegMap.Rmw8(reg, mask, valueShifted);

        // 寫完立刻回讀，讓 UI 與暫存器一致
        TryRefreshFromRegister();

        System.Diagnostics.Debug.WriteLine($"[GrayReverse] Set OK: Axis={Axis}, Enable={Enable}");
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[GrayReverse] Write failed: " + ex.Message);
    }
}