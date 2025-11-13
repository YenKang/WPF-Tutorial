using Newtonsoft.Json;
using Microsoft.Win32;
using System.IO;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;

private void ConfirmAndClose(object sender, RoutedEventArgs e)
{
    var vm = this.DataContext as IconToImageMapViewModel;
    if (vm == null)
    {
        MessageBox.Show("ViewModel 尚未初始化。",
            "錯誤", MessageBoxButton.OK, MessageBoxImage.Error);
        return;
    }

    // 1. 建立 JSON 物件
    IconOSDExportRoot exportRoot = vm.BuildExportModel();

    // 2. SaveFileDialog
    var sfd = new SaveFileDialog();
    sfd.Title = "儲存 JSON";
    sfd.FileName = "IconOSDMap.json";
    sfd.Filter = "JSON Files (*.json)|*.json";

    bool? ok = sfd.ShowDialog();
    if (ok != true)
        return;

    try
    {
        // 3. 序列化寫檔
        string json = JsonConvert.SerializeObject(exportRoot, Formatting.Indented);
        File.WriteAllText(sfd.FileName, json);

        MessageBox.Show("輸出完成！", "OK");
        this.DialogResult = true;
        this.Close();
    }
    catch (System.Exception ex)
    {
        MessageBox.Show("寫入失敗：" + ex.Message);
    }
}