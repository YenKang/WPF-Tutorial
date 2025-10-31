private void ResizeOrders(int desiredCount)
{
    desiredCount = Clamp(desiredCount, 1, 22);
    while (Orders.Count < desiredCount)
        Orders.Add(new OrderSlot { DisplayNo = Orders.Count + 1, SelectedIndex = 0 });

    while (Orders.Count > desiredCount)
        Orders.RemoveAt(Orders.Count - 1);

    for (int i = 0; i < Orders.Count; i++)
        Orders[i].DisplayNo = i + 1;
}