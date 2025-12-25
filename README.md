private async void Generate()
{
    var folder = SelectOutputFolder();
    if (string.IsNullOrWhiteSpace(folder))
        return;

    OutputFolder = folder;

    try
    {
        await System.Threading.Tasks.Task.Run(() =>
        {
            ImageExporter.ExportNoiseBmpBySizeGrid(
                HStart, HEnd, HStep,
                VStart, VEnd, VStep,
                OutputFolder,
                deterministicBySize: true
            );
        });

        System.Windows.MessageBox.Show("圖片輸出完成");
    }
    catch (Exception ex)
    {
        System.Windows.MessageBox.Show(ex.Message, "Export failed");
    }
}


＝＝＝＝＝

private string SelectOutputFolder()
{
    using (var dlg = new FolderBrowserDialog())
    {
        dlg.Description = "Select output folder";
        dlg.UseDescriptionForTitle = true;

        if (!string.IsNullOrWhiteSpace(OutputFolder))
            dlg.SelectedPath = OutputFolder;

        var result = dlg.ShowDialog();
        if (result != DialogResult.OK || string.IsNullOrWhiteSpace(dlg.SelectedPath))
            return null;

        return dlg.SelectedPath;
    }
}
