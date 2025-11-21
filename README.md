## Step 1：建立 BWLoopModel.cs

```
// Models/BWLoopModel.cs
namespace BistMode.Models
{
    public sealed class BWLoopModel
    {
        // UI 要調整的數值（最後會寫進暫存器）
        public int Fcnt2 { get; set; }

        // 要寫的暫存器名稱，例如 "BIST_FCNT2"
        public string RegName { get; set; }
    }
}
```

## Step 2：Pattern 裡要有一顆 BWLoopModel in BistodeViewModel.cs

```
// Pattern.cs
using BistMode.Models;

public sealed class Pattern
{
    public int Index { get; set; }
    public string Name { get; set; }

    // 這張圖對應的 BWLoop model（如果有 BWLoop 控制）
    public BWLoopModel BWLoop { get; set; }
}
```

## Step3:在 LoadFileFrom() 裡，從 JObject bwLoopControl → 填 BWLoopModel in BWLoopVM.cs


```
"BWLoopControl": {
  "register": "BIST_FCNT2",
  "default": "03E"
}
```


## step4: BistModeViewModel.LoadFileFrom() 裡，對每一個 pattern node 做解析時，多加一段「解析 BWLoopControl」：


```
private void LoadFileFrom(string profilePath)
{
    var json = File.ReadAllText(profilePath);
    var root = JObject.Parse(json);

    var patterns = root["patterns"] as JArray;
    if (patterns == null) return;

    PatternList.Clear();

    foreach (var item in patterns.OfType<JObject>())
    {
        var p = new Pattern();
        p.Index = item["index"] != null ? item["index"].Value<int>() : 0;
        p.Name  = (string)item["name"] ?? string.Empty;

        // ⭐ 這一段是 BWLoop 的解析
        var bwLoopControl = item["BWLoopControl"] as JObject;
        if (bwLoopControl != null)
        {
            // 1) 暫存器名字（例如 "BIST_FCNT2"）
            var regName = (string)bwLoopControl["register"] ?? string.Empty;

            // 2) default 值（例如 "03E"）
            var defText = (string)bwLoopControl["default"] ?? "0";
            int defValue = ParseHex(defText, 0);   // 轉成 int

            p.BWLoop = new BWLoopModel
            {
                RegName = regName,
                Fcnt2   = defValue
            };
        }

        PatternList.Add(p);
    }
}

// 小工具：把 "03E"/"0x03E" → int
private static int ParseHex(string text, int fallback)
{
    if (string.IsNullOrWhiteSpace(text)) return fallback;

    text = text.Trim();
    if (text.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
        text = text.Substring(2);

    int value;
    if (int.TryParse(text,
                     System.Globalization.NumberStyles.HexNumber,
                     null,
                     out value))
    {
        return value;
    }

    return fallback;
}
```


## Step5: 建立實體: 所以 ApplyPatternToGroups 裡 BWLoop 這一段要改成：


```
private void ApplyPatternToGroups(Pattern p)
{
    // BWLoop
    if (p.BWLoop != null)
    {
        IsBWLoopVisible = true;
        BWLoopVM.SetModel(p.BWLoop);   // ⬅ 改成傳「model 實體」
    }
    else
    {
        IsBWLoopVisible = false;
        BWLoopVM.SetModel(null);       // 避免殘留舊值（可加可不加，看你習慣）
    }

    // GrayLevel / Cursor 之後也會用一樣的 pattern：
    // if (p.GrayLevel != null) GrayLevelVM.SetModel(p.GrayLevel);
    // if (p.Cursor    != null) CursorVM.SetModel(p.Cursor);
}
```


## Step6: BWLoopVM.cs


```
using BistMode.Models;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class BWLoopVM : ViewModelBase
    {
        private BWLoopModel _model;
        private IRegisterReadWriteEx _regControl;

        public BWLoopVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        public void SetRegisterControl(IRegisterReadWriteEx regControl)
        {
            _regControl = regControl;
        }

        // ⭐ VM 接 Model 實體
        public void SetModel(BWLoopModel model)
        {
            _model = model;

            if (_model == null)
            {
                Fcnt2 = 0;
                RegName = string.Empty;
                return;
            }

            // 反映 Model 的值到 UI
            Fcnt2   = _model.Fcnt2;
            RegName = _model.RegName;
        }

        // ⭐ UI 綁定用
        public int Fcnt2
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;

                if (_model != null)
                    _model.Fcnt2 = value;     // ⭐ 回寫 Model（核心）
            }
        }

        // ⭐ 寫暫存器需要
        public string RegName
        {
            get { return GetValue<string>(); }
            private set { SetValue(value); }
        }

        public IRelayCommand ApplyCommand { get; }

        // ⭐ 寫入暫存器（使用 model.RemName）
        private void ApplyWrite()
        {
            if (_model == null) return;
            if (_regControl == null) return;
            if (string.IsNullOrEmpty(RegName)) return;

            uint raw = (uint)_model.Fcnt2;

            // 寫入暫存器（例如 BIST_FCNT2）
            _regControl.SetRegisterByName(RegName, raw);
        }
    }
}
```


