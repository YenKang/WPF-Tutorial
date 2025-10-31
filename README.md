public void Apply()
{
    if (Total <= 0) return;

    // Step 1: TOTAL（硬體 0 = 1 張）
    if (!string.IsNullOrEmpty(_totalTarget))
    {
        var encoded = Math.Max(0, Total - 1) & 0x1F;
        RegMap.Write8(_totalTarget, (byte)encoded);
    }

    // Step 2: ORD0..（只寫到 Orders 現有數量或 JSON targets 陣列數量上限）
    var count = Math.Min(Total, _orderTargets.Length);
    for (int i = 0; i < count; i++)
    {
        var target = _orderTargets[i];
        if (string.IsNullOrEmpty(target)) continue;

        byte code = (byte)(Orders[i].SelectedIndex & 0x3F);
        RegMap.Write8(target, code);
    }

    // Step 2.5: FCNT1（需先在 LoadFrom 解析 mapping）
    int f1Val = PackFcnt1FromUI(); // 10-bit
    WriteFcnt(_fc1LowTarget, _fc1HighTarget, _fc1Mask, _fc1Shift, f1Val);

    // Step 3: 觸發 Auto-Run（若有）
    if (!string.IsNullOrEmpty(_triggerTarget))
        RegMap.Write8(_triggerTarget, _triggerValue);

    // Step 4: 存 Cache（Total, Orders, FCNT1）
    var orderIdx = new List<int>(Orders.Count);
    for (int i = 0; i < Orders.Count; i++) orderIdx.Add(Orders[i].SelectedIndex);
    AutoRunCache.Save(Total, orderIdx, f1Val);

    System.Diagnostics.Debug.WriteLine("[AutoRun] Apply done. Cache updated.");
}