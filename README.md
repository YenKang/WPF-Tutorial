using Microsoft.Win32;
using System.Windows;

private void OnSaveIni(object sender, RoutedEventArgs e)
{
    var vm = (IconToImageMapViewModel)DataContext;
    var dlg = new SaveFileDialog
    {
        Filter = "INI files (*.ini)|*.ini|All files (*.*)|*.*",
        FileName = "ImageMapConfig.ini"
    };
    if (dlg.ShowDialog(this) == true)
    {
        vm.SaveToIni(dlg.FileName);
        MessageBox.Show("Saved.", "INI");
    }
}

private void OnLoadIni(object sender, RoutedEventArgs e)
{
    var vm = (IconToImageMapViewModel)DataContext;
    var dlg = new OpenFileDialog
    {
        Filter = "INI files (*.ini)|*.ini|All files (*.*)|*.*",
        FileName = "ImageMapConfig.ini"
    };
    if (dlg.ShowDialog(this) == true)
    {
        vm.LoadFromIni(dlg.FileName);
        MessageBox.Show("Loaded.", "INI");
    }
}