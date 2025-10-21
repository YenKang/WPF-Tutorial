public void Apply()
{
    // 1) 寫入總數：硬體定義 = (張數 - 1)
    if (!string.IsNullOrEmpty(_totalTarget))
    {
        var encoded = Math.Max(0, Total - 1) & TotalValueMask; // 確保不為負
        RegMap.Write8(_totalTarget, (byte)encoded);
    }

    // 2) 寫入 ORD0..ORD(Total-1)（這段不用改）
    var count = Math.Min(Total, _orderTargets.Length);
    for (int i = 0; i < count; i++)
    {
        var target = _orderTargets[i];
        if (string.IsNullOrEmpty(target)) continue;

        var code = (byte)(Orders[i].SelectedIndex & OrdValueMask);
        RegMap.Write8(target, code);
    }

    // 3) 觸發（不變）
    if (!string.IsNullOrEmpty(_triggerTarget))
        RegMap.Write8(_triggerTarget, _triggerValue);
}