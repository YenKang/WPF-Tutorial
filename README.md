private Dictionary<int, OSDIconFlashMap.Model.ImageOption> _currentMap = new();

private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    var win = new OSDIconFlashMap.View.IconToImageMapWindow();
    var owner = System.Windows.Window.GetWindow(this as System.Windows.DependencyObject);
    if (owner != null) win.Owner = owner;

    if (win.ShowDialog() == true)
    {
        _currentMap = win.ResultMap;
        System.Diagnostics.Debug.WriteLine($"Mapping count: {_currentMap.Count}");
    }
}