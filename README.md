using System.Windows.Data; // for CollectionViewSource

private bool _isHydrating; // 全域旗標，避免水球效應

private void ResizeOrders(int desired)
{
    desired = Clamp(desired, 1, 22);
    var view = CollectionViewSource.GetDefaultView(Orders);

    using (view?.DeferRefresh())
    {
        _isHydrating = true;

        // 先補到 desired 個（只放空殼，不賦值）
        while (Orders.Count < desired)
            Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1 });

        // 削到 desired
        while (Orders.Count > desired)
            Orders.RemoveAt(Orders.Count - 1);

        // 重新編號（避免中間刪除後號碼亂掉）
        for (int i = 0; i < Orders.Count; i++)
            Orders[i].DisplayNo = i + 1;

        _isHydrating = false;
    }
}