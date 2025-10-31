private void RestoreFromCache()
{
    // Total + Orders
    Total = Clamp(AutoRunCache.Total, 1, 22);
    ResizeOrders(Total);

    for (int i = 0; i < Orders.Count; i++)
        Orders[i].SelectedIndex = AutoRunCache.Orders != null && i < AutoRunCache.Orders.Length
            ? (AutoRunCache.Orders[i] & 0x3F)
            : 0;

    // FCNT1
    UnpackFcnt1ToUI(AutoRunCache.Fcnt1);
}