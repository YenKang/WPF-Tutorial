private void BuildTotalOptions(int min, int max)
{
    _totalMin = Clamp(min, 1, MaxOrders);
    _totalMax = Clamp(max, 1, MaxOrders);
    if (_totalMin > _totalMax) { var t = _totalMin; _totalMin = _totalMax; _totalMax = t; }

    TotalOptions.Clear();
    for (int v = _totalMin; v <= _totalMax; v++) TotalOptions.Add(v);
    System.Diagnostics.Debug.WriteLine($"[BuildTotalOptions] built {_totalMin}-{_totalMax}, count={TotalOptions.Count}");
}