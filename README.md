<Button Content="BIST Preset (右鍵編輯)"
        Margin="4"
        Command="{Binding RunPresetCommand}">
    <Button.InputBindings>
        <!-- 右鍵：開啟文字編輯器 -->
        <MouseBinding MouseAction="RightClick"
                      Command="{Binding EditPresetCommand}" />
    </Button.InputBindings>
</Button>


=====

,,,
using System;
using System.Diagnostics;
using System.IO;
using System.Globalization;
using System.Windows.Input;

public class BistModeViewModel : ViewModelBase
{
    // TODO: 這裡改成你實際的檔案路徑
    private readonly string _presetScriptPath =
        @"D:\BistScripts\BIST_Preset.py";

    public ICommand RunPresetCommand  { get; private set; }
    public ICommand EditPresetCommand { get; private set; }

    public BistModeViewModel()
    {
        RunPresetCommand  = new RelayCommand(_ => ExecutePresetScript());
        EditPresetCommand = new RelayCommand(_ => OpenPresetInEditor());
    }

    // 🟠 右鍵：打開 python 檔給你編輯
    private void OpenPresetInEditor()
    {
        try
        {
            var psi = new ProcessStartInfo
            {
                FileName = _presetScriptPath,
                UseShellExecute = true   // 用系統預設程式開啟 .py
            };
,
            Process.Start(psi);
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

    // 下面是左鍵的「先做一半」版本
    // ...
}
,,,


