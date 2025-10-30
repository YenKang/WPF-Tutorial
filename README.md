// ===== 來源決策 =====
if (_preferHw)
{
    // 偏好硬體：直接回讀，完全跳過 JSON default
    RefreshFromHardware();
}
else
{
    // JSON 模式：以 default 初始化
    int def = Clamp(totalObj?["default"]?.Value<int>() ?? MaxOrders, min, max);

    _isHydrating = true;
    try
    {
        // 設定 Total（只調數量；由 setter 內的 ResizeOrders 負責）
        Total = def;

        // 以 JSON default 回填全部 slot
        HydrateOrdersFromJsonDefaultsForRange(0, def);
    }
    finally
    {
        _isHydrating = false;
    }
}