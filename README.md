# Cursor

## step1: Model

- 建立 autorunModel.cs（放在 Models 資料夾）

```
using System.Collections.Generic;

namespace BistMode.Models
{
    public sealed class AutoRunModel
    {
        // Total
        public string TotalReg { get; set; }
        public int Total { get; set; }
        public int TotalMin { get; set; }
        public int TotalMax { get; set; }

        // Dynamic pattern order (N items from JSON)
        public List<string> OrderRegs { get; } = new List<string>();
        public List<int> OrderValues { get; } = new List<int>();

        // fcnt1 (no min/max)
        public string Fcnt1Reg { get; set; }
        public int Fcnt1 { get; set; }

        // fcnt2 (no min/max)
        public string Fcnt2Reg { get; set; }
        public int Fcnt2 { get; set; }
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

## Step 3：在 LoadFileFrom() 解析 autorun JSON → CursorModel

```json
"autoRunControl": {
  "title": "Auto Run",

  "total": {
    "register": "BIST_PT_TOTAL",
    "min": 1,
    "max": 22,
    "default": 22
  },

  "patternOrder": {
    "register": {
      "BIST_PT_ORD0": 0,
      "BIST_PT_ORD1": 1,
      "BIST_PT_ORD2": 2,
      "BIST_PT_ORD3": 3,
      "BIST_PT_ORD4": 4,
      "BIST_PT_ORD5": 5,
      "BIST_PT_ORD6": 6,
      "BIST_PT_ORD7": 7,
      "BIST_PT_ORD8": 8,
      "BIST_PT_ORD9": 9,
      "BIST_PT_ORD10": 10,
      "BIST_PT_ORD11": 11,
      "BIST_PT_ORD12": 12,
      "BIST_PT_ORD13": 13,
      "BIST_PT_ORD14": 14,
      "BIST_PT_ORD15": 15,
      "BIST_PT_ORD16": 16,
      "BIST_PT_ORD17": 17,
      "BIST_PT_ORD18": 18,
      "BIST_PT_ORD19": 19,
      "BIST_PT_ORD20": 20,
      "BIST_PT_ORD21": 21
    }
  },

  "fcnt1": {
    "register": "BIST_FCNT1",
    "min": 30,
    "max": 600,
    "default": 60
  },

  "fcnt2": {
    "register": "BIST_FCNT2",
    "min": 30,
    "max": 480,
    "default": 30
  }
}
```
### Csharp Code for Parsing Json Node

```
//-----------------------------------------------------------
// AutoRun JSON → AutoRunModel
//-----------------------------------------------------------
var autoRunNode = pnode["autoRunControl"] as JObject;
if (autoRunNode != null)
{
    var m = new AutoRunModel();
    Debug.WriteLine("=== [AutoRun JSON PARSE] Start ===");

    // ---------------- total ----------------
    m.TotalReg = autoRunNode["total"]?["register"]?.Value<string>() ?? "";
    m.TotalMin = autoRunNode["total"]?["min"]?.Value<int>() ?? 1;
    m.TotalMax = autoRunNode["total"]?["max"]?.Value<int>() ?? 22;
    m.Total    = autoRunNode["total"]?["default"]?.Value<int>() ?? m.TotalMin;

    Debug.WriteLine($"[AutoRun] TotalReg={m.TotalReg}, Total={m.Total}");

    // ---------------- patternOrder (動態 N 筆) ----------------
    m.OrderRegs.Clear();
    m.OrderValues.Clear();

    var regObj = autoRunNode["patternOrder"]?["register"] as JObject;
    if (regObj != null)
    {
        foreach (var prop in regObj.Properties())
        {
            var regName    = prop.Name;
            var defaultVal = (int)prop.Value; // (int)JToken 最乾淨寫法

            m.OrderRegs.Add(regName);
            m.OrderValues.Add(defaultVal);

            Debug.WriteLine($"[AutoRun] OrderReg={regName}, Default={defaultVal}");
        }
    }

    // ---------------- fcnt1 ----------------
    m.Fcnt1Reg = autoRunNode["fcnt1"]?["register"]?.Value<string>() ?? "";
    m.Fcnt1    = autoRunNode["fcnt1"]?["default"]?.Value<int>() ?? 0;
    Debug.WriteLine($"[AutoRun] fcnt1 {m.Fcnt1Reg} = {m.Fcnt1}");

    // ---------------- fcnt2 ----------------
    m.Fcnt2Reg = autoRunNode["fcnt2"]?["register"]?.Value<string>() ?? "";
    m.Fcnt2    = autoRunNode["fcnt2"]?["default"]?.Value<int>() ?? 0;
    Debug.WriteLine($"[AutoRun] fcnt2 {m.Fcnt2Reg} = {m.Fcnt2}");

    // ---------------- 存入 Pattern ----------------
    p.AutoRunModel = m;

    Debug.WriteLine("=== [AutoRun JSON PARSE] Done ===");
}
```

## step4: autoRunVM.cs

```
using System;
using System.Collections.ObjectModel;
using System.Diagnostics;
using BistMode.Models;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private AutoRunModel _model;
        private IRegisterControl _reg;

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);

            // OrderValues (UI 用)
            Orders = new ObservableCollection<int>();
        }

        // ----------------------------------------------------
        // RegisterControl
        // ----------------------------------------------------
        public void SetRegisterControl(IRegisterControl r)
        {
            _reg = r;
        }

        // ----------------------------------------------------
        // Model 綁定
        // ----------------------------------------------------
        public void SetModel(AutoRunModel m)
        {
            _model = m;

            if (m == null)
            {
                Total = 0;
                Orders.Clear();
                Fcnt1 = 0;
                Fcnt2 = 0;
                Debug.WriteLine("[AutoRun.SetModel] NULL model");
                return;
            }

            // total
            TotalMin = m.TotalMin;
            TotalMax = m.TotalMax;
            Total    = m.Total;

            // Orders
            Orders.Clear();
            foreach (var v in m.OrderValues)
                Orders.Add(v);

            // fcnt1 / fcnt2
            Fcnt1 = m.Fcnt1;
            Fcnt2 = m.Fcnt2;

            Debug.WriteLine("[AutoRun.SetModel]");
            Debug.WriteLine($" Total={Total}");
            Debug.WriteLine($" Fcnt1={Fcnt1}  Fcnt2={Fcnt2}");
            Debug.WriteLine(" Order values: " + string.Join(", ", Orders));
        }

        // ----------------------------------------------------
        // Properties for Binding
        // ----------------------------------------------------

        // Total
        public int Total
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                    _model.Total = value;
            }
        }

        public int TotalMin
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        public int TotalMax
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        // Orders (22 items)
        public ObservableCollection<int> Orders { get; }

        // Fcnt1
        public int Fcnt1
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                    _model.Fcnt1 = value;
            }
        }

        // Fcnt2
        public int Fcnt2
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                    _model.Fcnt2 = value;
            }
        }

        public IRelayCommand ApplyCommand { get; }

        // ----------------------------------------------------
        // ApplyWrite → 寫暫存器
        // ----------------------------------------------------
        private void ApplyWrite()
        {
            if (_model == null || _reg == null)
            {
                Debug.WriteLine("[AutoRun.ApplyWrite] FAIL: model or reg == null");
                return;
            }

            Debug.WriteLine("=== [AutoRun.ApplyWrite] Start ===");

            // total
            if (!string.IsNullOrEmpty(_model.TotalReg))
            {
                _reg.SetRegisterByName(_model.TotalReg, (uint)_model.Total);
                Debug.WriteLine($" Write { _model.TotalReg } = { _model.Total }");
            }

            // orders
            for (int i = 0; i < _model.OrderRegs.Count; i++)
            {
                string regName = _model.OrderRegs[i];
                int val = Orders[i];
                _model.OrderValues[i] = val; // keep model updated

                _reg.SetRegisterByName(regName, (uint)val);
                Debug.WriteLine($" Write {regName} = {val}");
            }

            // fcnt1
            if (!string.IsNullOrEmpty(_model.Fcnt1Reg))
            {
                _reg.SetRegisterByName(_model.Fcnt1Reg, (uint)_model.Fcnt1);
                Debug.WriteLine($" Write { _model.Fcnt1Reg } = { _model.Fcnt1:X2 }");
            }

            // fcnt2
            if (!string.IsNullOrEmpty(_model.Fcnt2Reg))
            {
                _reg.SetRegisterByName(_model.Fcnt2Reg, (uint)_model.Fcnt2);
                Debug.WriteLine($" Write { _model.Fcnt2Reg } = { _model.Fcnt2:X2 }");
            }

            Debug.WriteLine("=== [AutoRun.ApplyWrite] Done ===");
        }
    }
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
