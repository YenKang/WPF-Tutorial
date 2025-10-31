private void RestoreFromCache()
    {
        _isHydrating = true;
        try
        {
            // 回填 Total 與 Orders
            Total = Clamp(AutoRunCache.Total, TotalMinDefault, TotalMaxDefault);
            ResizeOrders(Total);

            for (int i = 0; i < Orders.Count && i < AutoRunCache.OrderIndices.Length; i++)
                Orders[i].SelectedIndex = CoercePatternIndex(AutoRunCache.OrderIndices[i]);

            // 回填 FCNT1（若有）
            UnpackFcnt1ToUI(AutoRunCache.Fcnt1);
        }
        finally
        {
            _isHydrating = false;