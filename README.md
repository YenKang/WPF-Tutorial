// 需要這兩個 using
using OSDIconFlashMap.Model;   // IconModel
using OSDIconFlashMap.View;    // IconFlashMapWindow

private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    // 假資料：icon_1 ~ icon_12，地址從 0x18000 起，每個間隔 0x02222（只是示意）
    const ulong baseAddr = 0x18000;
    const int   count    = 12;

    var icons = new List<IconModel>();
    for (int i = 1; i <= count; i++)
    {
        icons.Add(new IconModel
        {
            Id           = i,
            Name         = $"icon_{i}",
            FlashStart   = baseAddr + (ulong)((i - 1) * 0x2222),
            // 先不載入圖片，只放規則鍵名：iconN → iconN_thumbnail
            ThumbnailKey = $"icon_{i}_thumbnail"
        });
    }

    var win = new IconFlashMapWindow();
    win.LoadCatalog(icons);

    // 如果這個 method 不在 Window 類別內，用這招安全帶 Owner
    var owner = System.Windows.Window.GetWindow(this as System.Windows.DependencyObject);
    if (owner != null) win.Owner = owner;

    win.Show();
}