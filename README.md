private void ResizeOrders(int desiredCount)
{
    // 合法化目標數量
    desiredCount = Clamp(desiredCount, 1, 22);

    // 記下擴編前的數量
    int oldCount = Orders.Count;

    _isHydrating = true;
    try
    {
        // 1) 若數量不足 → 逐一補上
        while (Orders.Count < desiredCount)
        {
            int slot = Orders.Count;                // 0-based slot
            int idx  = CoercePatternIndex(slot);    // 需求：用 slot 番號 0,1,2,3… 補齊

            Orders.Add(new OrderSlot
            {
                DisplayNo     = slot + 1,           // 顯示用 1-based
                SelectedIndex = idx
            });
        }

        // 2) 若數量過多 → 刪除尾端
        while (Orders.Count > desiredCount)
            Orders.RemoveAt(Orders.Count - 1);

        // 3) 重新編排顯示用序號（不影響 SelectedIndex）
        for (int i = 0; i < Orders.Count; i++)
            Orders[i].DisplayNo = i + 1;
    }
    finally
    {
        _isHydrating = false;
    }
}