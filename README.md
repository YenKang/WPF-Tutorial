# Cursor

## step1: Model

- 建立 CursorModel.cs（放在 Models 資料夾）

```
// 檔案：Models/CursorModel.cs
namespace BistMode.Models
{
    public sealed class CursorModel
    {
        public bool CursorEnable { get; set; }
        public bool DiagonalEnable { get; set; }

        public int HPos { get; set; }
        public int VPos { get; set; }

        public string CursorEnableReg { get; set; }
        public string DiagonalEnableReg { get; set; }
        public string HPosReg { get; set; }
        public string VPosReg { get; set; }
    }
}
```

## Step 2：在 Pattern / PatternItem 裡加一個欄位

```
public sealed class Pattern
{
    public string Name { get; set; }

    // 已經有的
    public GrayLevelModel GrayLevelModel { get; set; }
    public BWLoopModel BWLoopModel { get; set; }
    public FlickerRGBModel FlickerRGBModel { get; set; }

    // 👉 新增這行
    public CursorModel CursorModel { get; set; }
}
```

## Step 3：在 LoadFileFrom() 解析 cursorControl JSON → CursorModel

```
"cursorControl":
{
    "diagonalEn":  false,
    "cursorEn":    false,
    "HPos":        960,
    "VPos":        360,

    "cursorEnReg":   "BIST_CURSOR_EN",
    "diagonalEnReg": "BIST_DIAGONAL_EN",
    "hPosReg":       "BIST_CURSOR_HPOS",
    "vPosReg":       "BIST_CURSOR_VPOS"
}
```
### Csharp Code for Parsing Json Node

```
var cursorNode = pnode["cursorControl"] as JObject;
if (cursorNode != null)
{
    var m = new CursorModel
    {
        CursorEnable     = cursorNode["cursorEn"]?.Value<bool>() ?? false,
        DiagonalEnable   = cursorNode["diagonalEn"]?.Value<bool>() ?? false,

        HPos             = cursorNode["HPos"]?.Value<int>() ?? 0,
        VPos             = cursorNode["VPos"]?.Value<int>() ?? 0,

        CursorEnableReg   = cursorNode["cursorEnReg"]?.Value<string>() ?? "",
        DiagonalEnableReg = cursorNode["diagonalEnReg"]?.Value<string>() ?? "",
        HPosReg           = cursorNode["hPosReg"]?.Value<string>() ?? "",
        VPosReg           = cursorNode["vPosReg"]?.Value<string>() ?? ""
    };

    p.CursorModel = m;
}
```

## step4: BistModeVM.cs 傳入 ApplyPatternToGroups(p) → 傳入 CursorModel

```
if (p.CursorModel != null)
{
    CursorVM.SetModel(p.CursorModel);
}
else
{
    CursorVM.SetModel(null);
}
```

## step5:CursorModel.cs

```
using System.Diagnostics;
using BistMode.Models;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class CursorVM : ViewModelBase
    {
        private CursorModel _model;
        private IRegisterControl _reg;

        public CursorVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        public void SetRegisterControl(IRegisterControl c)
        {
            _reg = c;
        }

        public void SetModel(CursorModel m)
        {
            _model = m;

            if (m == null)
            {
                IsCursorEnabled = false;
                IsDiagonalEnabled = false;
                HPos = 0;
                VPos = 0;
                return;
            }

            IsCursorEnabled   = m.CursorEnable;
            IsDiagonalEnabled = m.DiagonalEnable;
            HPos = m.HPos;
            VPos = m.VPos;

            Debug.WriteLine($"[Cursor.SetModel] CE={IsCursorEnabled}, DE={IsDiagonalEnabled}, H={HPos}, V={VPos}");
        }

        // Properties
        public bool IsCursorEnabled
        {
            get => GetValue<bool>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.CursorEnable = value;

                OnPropertyChanged(nameof(IsPosEnabled));
            }
        }

        public bool IsDiagonalEnabled
        {
            get => GetValue<bool>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.DiagonalEnable = value;
            }
        }

        public int HPos
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.HPos = value;
            }
        }

        public int VPos
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.VPos = value;
            }
        }

        public bool IsPosEnabled => IsCursorEnabled;

        public IRelayCommand ApplyCommand { get; }

        private void ApplyWrite()
        {
            if (_model == null || _reg == null) return;

            Debug.WriteLine("[Cursor.ApplyWrite]");
            Debug.WriteLine($" CE={_model.CursorEnable}, DE={_model.DiagonalEnable}, H={_model.HPos}, V={_model.VPos}");

            _reg.SetRegisterByName(_model.CursorEnableReg,   _model.CursorEnable ? 1u : 0u);
            _reg.SetRegisterByName(_model.DiagonalEnableReg, _model.DiagonalEnable ? 1u : 0u);
            _reg.SetRegisterByName(_model.HPosReg,           (uint)_model.HPos);
            _reg.SetRegisterByName(_model.VPosReg,           (uint)_model.VPos);

            Debug.WriteLine("[Cursor.ApplyWrite] DONE.");
        }
    }
}
```

## Cursor.xaml

```
<GroupBox Header="Cursor" Margin="0,12,0,12">
    <StackPanel Margin="12">

        <!-- Enable 區 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <CheckBox Content="Cursor Enable"
                      Margin="0,0,12,0"
                      IsChecked="{Binding CursorVM.IsCursorEnabled, Mode=TwoWay}"/>

            <CheckBox Content="Diagonal"
                      IsChecked="{Binding CursorVM.IsDiagonalEnabled, Mode=TwoWay}"/>
        </StackPanel>

        <!-- HPos -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="HPos" Width="60" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown Width="100"
                                Minimum="0"
                                Maximum="4095"
                                IsEnabled="{Binding CursorVM.IsPosEnabled}"
                                Value="{Binding CursorVM.HPos, Mode=TwoWay}"/>
        </StackPanel>

        <!-- VPos -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="VPos" Width="60" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown Width="100"
                                Minimum="0"
                                Maximum="4095"
                                IsEnabled="{Binding CursorVM.IsPosEnabled}"
                                Value="{Binding CursorVM.VPos, Mode=TwoWay}"/>
        </StackPanel>

        <Button Content="Set Cursor"
                Width="120" Height="28"
                HorizontalAlignment="Right"
                Command="{Binding CursorVM.ApplyCommand}"/>
    </StackPanel>
</GroupBox>
```
