private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    var win = new IconToImageMapWindow();
    var owner = Window.GetWindow(this as DependencyObject);
    if (owner != null) win.Owner = owner;

    // 建議：如果你還沒放 OK/Cancel 按鈕，先加（見步驟 2）
    if (win.ShowDialog() == true)
    {
        // ✔ 1) 用正確的欄位名（有底線）
        // ✔ 2) 保險：ResultMap 可能為 null
        _currentMap = win.ResultMap ?? new Dictionary<int, ImageOption>();

        System.Diagnostics.Debug.WriteLine($"Mapping count: {_currentMap.Count}");
    }
}