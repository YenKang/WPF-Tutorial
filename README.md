<Button Content="BIST Preset (左鍵執行 / 右鍵編輯)"
        Margin="6"
        Command="{Binding RunPresetCommand}">
    <Button.InputBindings>

        <!-- 🟠 右鍵：EditPresetCommand -->
        <MouseBinding MouseAction="RightClick"
                      Command="{Binding EditPresetCommand}" />

    </Button.InputBindings>
</Button>


public void ExecutePresetScript()
{
    if (!File.Exists(_presetScriptPath))
    {
        ShowMessage("找不到 preset 檔：\n" + _presetScriptPath);
        return;
    }

    string[] lines;

    try
    {
        lines = File.ReadAllLines(_presetScriptPath);
    }
    catch (Exception ex)
    {
        ShowMessage("讀取 preset 檔失敗：\n" + ex.Message);
        return;
    }

    // 🔵 一行一行顯示（先做這個，不做解析、不寫暫存器）
    string all = "";
    foreach (var raw in lines)
    {
        all += raw + "\n";     // 把每一行附加起來
    }

    ShowMessage(all);          // 跳出視窗顯示所有行
}