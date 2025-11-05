using OSDIconFlashMap.Model;
using OSDIconFlashMap.View;

private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    var icons = new List<IconModel>();

    // 假設你這裡手上就有前人流程產生的 Bitmap 與 address
    // (bmp1, bmp2, bmp3 只是示意，請換成你實際的變數)
    icons.Add(new IconModel{
        Id = 1, Name = "icon_1", FlashStart = 0x18000,
        Thumbnail = ShowUImg(bmp1),     // 直接吃你現有方法的回傳值
        RawBitmap = bmp1                // 可選
    });
    icons.Add(new IconModel{
        Id = 2, Name = "icon_2", FlashStart = 0x18222,
        Thumbnail = ShowUImg(bmp2),
        RawBitmap = bmp2
    });
    // … 依序加入

    var win = new IconFlashMapWindow();
    win.LoadCatalog(icons);
    win.Owner = this;
    win.Show();
}