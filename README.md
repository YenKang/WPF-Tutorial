## ApplyPatternToGroups 
```
if (p.BWLoopModel != null)
{
    IsBWLoopVisible = true;
    BWLoopVM.SetModel(p.BWLoopModel);
}
else
{
    IsBWLoopVisible = false;
    BWLoopVM.SetModel(null);
}
```

## Step2:

```
public void SetModel(BWLoopModel model)
{
    _model = model;

    if (_model == null)
    {
        // 沒有 model 時，把 UI 清掉
        Fcnt2    = 0;
        Fcnt2Min = 0;
        Fcnt2Max = 0;
        RegName  = string.Empty;
        return;
    }

    // Model → VM（把目前這張圖的狀態推到 UI）
    RegName  = _model.RegName;
    Fcnt2Min = _model.Fcnt2Min;
    Fcnt2Max = _model.Fcnt2Max;
    Fcnt2    = _model.Fcnt2;
}
```

## UI 變動 寫回model

```
public int Fcnt2
{
    get { return GetValue<int>(); }
    set
    {
        if (!SetValue(value)) return;

        // ⭐ 只要 UI 有變動，就寫回 Model
        if (_model != null)
            _model.Fcnt2 = value;
    }
}
```


## 寫值
```
private void ApplyWrite()
{
    if (_model == null) return;
    if (_regControl == null) return;
    if (string.IsNullOrEmpty(RegName)) return;

    uint raw = (uint)_model.Fcnt2;

    // ⭐ 真正寫暫存器，RegName 會是 "BIST_FCNT2"
    _regControl.SetRegisterByName(RegName, raw);
}
```


## 完整版

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

        // 給 BistModeViewModel 呼叫
        public void SetRegisterControl(IRegisterReadWriteEx regControl)
        {
            _regControl = regControl;
        }

        // ⭐ 核心：切換圖片時，把該圖的 Model 塞進來
        public void SetModel(BWLoopModel model)
        {
            _model = model;

            if (_model == null)
            {
                Fcnt2    = 0;
                Fcnt2Min = 0;
                Fcnt2Max = 0;
                RegName  = string.Empty;
                return;
            }

            // Model → VM
            RegName  = _model.RegName;
            Fcnt2Min = _model.Fcnt2Min;
            Fcnt2Max = _model.Fcnt2Max;
            Fcnt2    = _model.Fcnt2;
        }

        // ====== UI 綁定屬性 ======

        public int Fcnt2
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;

                // ⭐ UI → Model，即時同步
                if (_model != null)
                    _model.Fcnt2 = value;
            }
        }

        public int Fcnt2Min
        {
            get { return GetValue<int>(); }
            private set { SetValue(value); }
        }

        public int Fcnt2Max
        {
            get { return GetValue<int>(); }
            private set { SetValue(value); }
        }

        public string RegName
        {
            get { return GetValue<string>(); }
            private set { SetValue(value); }
        }

        public IRelayCommand ApplyCommand { get; }

        // ====== 寫入暫存器 ======
        private void ApplyWrite()
        {
            if (_model == null) return;
            if (_regControl == null) return;
            if (string.IsNullOrEmpty(RegName)) return;

            uint raw = (uint)_model.Fcnt2;

            // ⭐ 直接用 Model 裡的暫存器名稱
            _regControl.SetRegisterByName(RegName, raw);
        }
    }
}
```
