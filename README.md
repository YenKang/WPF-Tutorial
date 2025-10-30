private void SafeSetTotal(int v)
{
    v = Clamp(v, 1, MaxOrders);
    System.Diagnostics.Debug.WriteLine($"[SafeSetTotal] start v={v}, TotalOptions.Count={TotalOptions.Count}");

    if (TotalOptions.Count == 0)
    {
        System.Diagnostics.Debug.WriteLine("[SafeSetTotal] TotalOptions empty, defer via Dispatcher");
        System.Windows.Application.Current.Dispatcher.BeginInvoke(
            new Action(() =>
            {
                EnsureTotalOptionsCovers(v);
                Total = v;
                System.Diagnostics.Debug.WriteLine($"[SafeSetTotal-defer] applied Total={Total}");
            }),
            System.Windows.Threading.DispatcherPriority.Background
        );
        return;
    }

    EnsureTotalOptionsCovers(v);
    Total = v;
    System.Diagnostics.Debug.WriteLine($"[SafeSetTotal] done Total={Total}, TotalOptions.Count={TotalOptions.Count}");
}