public void RefreshFromRegister()
{
    var (reg, mask, shift) = GetAxisSpec(Axis);
    int raw = RegMap.Read8(reg);
    Enable = ((raw & mask) >> shift) != 0;

    System.Diagnostics.Debug.WriteLine($"[GrayReverse] Refresh: Axis={Axis}, Enable={Enable}");
}

private void TryRefreshFromRegister()
{
    try { RefreshFromRegister(); }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine("[GrayReverse] Skip refresh: " + ex.Message);
    }
}