public int Total
{
    get { return GetValue<int>(); }
    set
    {
        if (value < 1) value = 1;
        if (value > MaxOrders) value = MaxOrders;

        System.Diagnostics.Debug.WriteLine($"[Total setter] before SetValue={value}, TotalOptions.Count={TotalOptions.Count}");
        if (SetValue(value))
            ResizeOrders(value);
        System.Diagnostics.Debug.WriteLine($"[Total setter] after SetValue={Total}");
    }
}