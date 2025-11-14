using Microsoft.Win32;
using System.IO;

private void BtnLoadImages_Click(object sender, RoutedEventArgs e)
{
    var vm = this.DataContext as IconToImageMapViewModel;
    if (vm == null)
    {
        MessageBox.Show("ViewModel 尚未初始化。");
        return;
    }

    var dlg = new OpenFileDialog();
    dlg.Title = "選擇圖片檔案";
    dlg.Multiselect = true;
    dlg.Filter = "Image Files (*.bmp;*.jpg;*.jpeg;*.png)|*.bmp;*.jpg;*.jpeg;*.png";

    var result = dlg.ShowDialog(this);
    if (result != true)
        return;

    vm.LoadImagesFromFiles(dlg.FileNames);

    // 如想 Debug
    // Debug.WriteLine($"Loaded Images Count = {vm.Images.Count}");
}