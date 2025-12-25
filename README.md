private async void GenerateBmp_Click(object sender, RoutedEventArgs e)
{
    // 1) 讓使用者選輸出資料夾
    string outputDir = null;
    using (var dlg = new FolderBrowserDialog())
    {
        dlg.Description = "Select output folder for BMP images";
        dlg.UseDescriptionForTitle = true;

        var result = dlg.ShowDialog();
        if (result != System.Windows.Forms.DialogResult.OK || string.IsNullOrWhiteSpace(dlg.SelectedPath))
            return;

        outputDir = dlg.SelectedPath;
    }

    // 2) 取你的 UI 參數（你把這六個變數換成你實際的來源）
    //    如果你是 TextBox/UpDown，就在這裡讀值；如果你用 VM，就從 DataContext 讀
    int hStart = HStart;
    int hEnd   = HEnd;
    int hStep  = HStep;

    int vStart = VStart;
    int vEnd   = VEnd;
    int vStep  = VStep;

    // 3) 背景執行避免 UI 卡住
    try
    {
        await Task.Run(() =>
        {
            ImageExporter.ExportNoiseBmpBySizeGrid(
                hStart: hStart, hEnd: hEnd, hStep: hStep,
                vStart: vStart, vEnd: vEnd, vStep: vStep,
                outputDir: outputDir,
                deterministicBySize: true
            );
        });

        MessageBox.Show("BMP export done!");
    }
    catch (System.Exception ex)
    {
        MessageBox.Show(ex.Message, "Export failed");
    }
}
