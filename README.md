using System;
using System.IO;
// 如果你願意，也可以在這裡加：using System.Diagnostics;

...

private readonly string _presetScriptPath =
    @"D:\1.AOS\1064_Bist\tddi_engtool\BIST_Preset.py";  // ← 路徑自己改

public void OpenPresetInEditor()
{
    try
    {
        // 先確認檔案存在，避免一按就丟例外
        if (!File.Exists(_presetScriptPath))
        {
            ShowMessage("找不到 preset 檔：\n" + _presetScriptPath);
            return;
        }

        var psi = new System.Diagnostics.ProcessStartInfo()
        {
            FileName = _presetScriptPath,
            UseShellExecute = true   // 用系統預設程式開啟 .py
        };

        System.Diagnostics.Process.Start(psi);
    }
    catch (Exception ex)
    {
        ShowMessage("開啟 preset 檔失敗：\n" + ex.Message);
    }
}

private void ShowMessage(string msg)
{
    System.Windows.MessageBox.Show(msg, "BIST Preset");
}