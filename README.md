using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Collections.ObjectModel;

public class BistModeViewModel : INotifyPropertyChanged
{
    // 右側：直接綁定 XAML ItemsSource
    private List<ConfigUIRaw> _configUiRaws = new List<ConfigUIRaw>();
    public List<ConfigUIRaw> ConfigUiRaws
    {
        get { return _configUiRaws; }
        set { _configUiRaws = value; OnPropertyChanged("ConfigUiRaws"); }
    }

    public BistModeViewModel()
    {
        // ✅ 手動建立一筆假資料
        ConfigUiRaws = new List<ConfigUIRaw>
        {
            new ConfigUIRaw
            {
                title = "GrayLevel [9:0]",
                kind = "grayLevelControl",
                fields = new Dictionary<string, FieldDef>
                {
                    ["D2"] = new FieldDef { min = 0, max = 3,  @default = 2 },
                    ["D1"] = new FieldDef { min = 0, max = 15, @default = 8 },
                    ["D0"] = new FieldDef { min = 0, max = 15, @default = 12 }
                }
            }
        };
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void OnPropertyChanged(string name)
    {
        var h = PropertyChanged; if (h != null) h(this, new PropertyChangedEventArgs(name));
    }
}

// 以下是你既有 Model 的最小必要部分（可放同檔）
public class ConfigUIRaw
{
    public string title { get; set; }
    public string kind { get; set; }
    public Dictionary<string, FieldDef> fields { get; set; } = new Dictionary<string, FieldDef>();
}

public class FieldDef
{
    public int min { get; set; }
    public int max { get; set; }
    public int @default { get; set; }
}