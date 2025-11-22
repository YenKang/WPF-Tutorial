using System.Diagnostics;
using Utility.MVVM.Command;

public sealed class FlickerRGBVM : ViewModelBase
{
    private FlickerRGBModel _model;
    private IRegisterControl _regControl;

    public FlickerRGBVM()
    {
        ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
    }

    public void SetRegisterControl(IRegisterControl control)
    {
        _regControl = control;
    }

    // ---------------------------------------------------------
    //  SetModel (加入 Debug 訊息)
    // ---------------------------------------------------------
    public void SetModel(FlickerRGBModel model)
    {
        _model = model;

        Debug.WriteLine("=== [FlickerRGBVM] SetModel() ===");
        Debug.WriteLine(
            $"Page1 = ({model.RPage1:X2}, {model.GPage1:X2}, {model.BPage1:X2})");
        Debug.WriteLine(
            $"Page2 = ({model.RPage2:X2}, {model.GPage2:X2}, {model.BPage2:X2})");
        Debug.WriteLine(
            $"Page3 = ({model.RPage3:X2}, {model.GPage3:X2}, {model.BPage3:X2})");
        Debug.WriteLine(
            $"Page4 = ({model.RPage4:X2}, {model.GPage4:X2}, {model.BPage4:X2})");
        Debug.WriteLine("==================================");

        // 對應到 12 個 VM Property
        RPage1 = model.RPage1;
        GPage1 = model.GPage1;
        BPage1 = model.BPage1;

        RPage2 = model.RPage2;
        GPage2 = model.GPage2;
        BPage2 = model.BPage2;

        RPage3 = model.RPage3;
        GPage3 = model.GPage3;
        BPage3 = model.BPage3;

        RPage4 = model.RPage4;
        GPage4 = model.GPage4;
        BPage4 = model.BPage4;
    }


    // ---------------------------------------------------------
    //  ApplyWrite (加入 Page1~4 簡潔 Debug)
    // ---------------------------------------------------------
    private void ApplyWrite()
    {
        if (_model == null || _regControl == null)
            return;

        // 寫入 Page1
        _regControl.SetRegisterByName(_model.RPage1Reg, (uint)_model.RPage1);
        _regControl.SetRegisterByName(_model.GPage1Reg, (uint)_model.GPage1);
        _regControl.SetRegisterByName(_model.BPage1Reg, (uint)_model.BPage1);

        // Page2
        _regControl.SetRegisterByName(_model.RPage2Reg, (uint)_model.RPage2);
        _regControl.SetRegisterByName(_model.GPage2Reg, (uint)_model.GPage2);
        _regControl.SetRegisterByName(_model.BPage2Reg, (uint)_model.BPage2);

        // Page3
        _regControl.SetRegisterByName(_model.RPage3Reg, (uint)_model.RPage3);
        _regControl.SetRegisterByName(_model.GPage3Reg, (uint)_model.GPage3);
        _regControl.SetRegisterByName(_model.BPage3Reg, (uint)_model.BPage3);

        // Page4
        _regControl.SetRegisterByName(_model.RPage4Reg, (uint)_model.RPage4);
        _regControl.SetRegisterByName(_model.GPage4Reg, (uint)_model.GPage4);
        _regControl.SetRegisterByName(_model.BPage4Reg, (uint)_model.BPage4);

        // Debug：一頁一行
        Debug.WriteLine("=== [FlickerRGBVM] ApplyWrite() ===");
        Debug.WriteLine(
            $"Page1 = ({_model.RPage1:X2}, {_model.GPage1:X2}, {_model.BPage1:X2})");
        Debug.WriteLine(
            $"Page2 = ({_model.RPage2:X2}, {_model.GPage2:X2}, {_model.BPage2:X2})");
        Debug.WriteLine(
            $"Page3 = ({_model.RPage3:X2}, {_model.GPage3:X2}, {_model.BPage3:X2})");
        Debug.WriteLine(
            $"Page4 = ({_model.RPage4:X2}, {_model.GPage4:X2}, {_model.BPage4:X2})");
        Debug.WriteLine("==================================");
    }


    // ---------------------------------------------------------
    //  Properties: VM ↔ Model 連動 (保持你原本風格)
    // ---------------------------------------------------------
    public int RPage1 { get => GetValue<int>(); set { if (SetValue(value)) _model.RPage1 = value; } }
    public int GPage1 { get => GetValue<int>(); set { if (SetValue(value)) _model.GPage1 = value; } }
    public int BPage1 { get => GetValue<int>(); set { if (SetValue(value)) _model.BPage1 = value; } }

    public int RPage2 { get => GetValue<int>(); set { if (SetValue(value)) _model.RPage2 = value; } }
    public int GPage2 { get => GetValue<int>(); set { if (SetValue(value)) _model.GPage2 = value; } }
    public int BPage2 { get => GetValue<int>(); set { if (SetValue(value)) _model.BPage2 = value; } }

    public int RPage3 { get => GetValue<int>(); set { if (SetValue(value)) _model.RPage3 = value; } }
    public int GPage3 { get => GetValue<int>(); set { if (SetValue(value)) _model.GPage3 = value; } }
    public int BPage3 { get => GetValue<int>(); set { if (SetValue(value)) _model.BPage3 = value; } }

    public int RPage4 { get => GetValue<int>(); set { if (SetValue(value)) _model.RPage4 = value; } }
    public int GPage4 { get => GetValue<int>(); set { if (SetValue(value)) _model.GPage4 = value; } }
    public int BPage4 { get => GetValue<int>(); set { if (SetValue(value)) _model.BPage4 = value; } }

    public IRelayCommand ApplyCommand { get; }
}
