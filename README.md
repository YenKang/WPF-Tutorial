using OSDIconFlashMap.Model;
using OSDIconFlashMap.View;
using System.Collections.Generic;

private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    var icons = new List<IconModel>
    {
        new IconModel{ Id=1, Name="icon_1", FlashStart=0x18000 },
        new IconModel{ Id=2, Name="icon_2", FlashStart=0x18222 },
        new IconModel{ Id=3, Name="icon_3", FlashStart=0x18880 },
    };

    var win = new IconFlashMapWindow();
    win.LoadCatalog(icons);
    win.Show();
}