## FlickerRGBVM.vs

```
using BistMode.Models;
using Utility.MVVM;
using Utility.MVVM.Command;

namespace BistMode.ViewModels
{
    public sealed class FlickerRGBVM : ViewModelBase
    {
        private FlickerRGBModel _model;
        private IRegisterReadWriteEx _regControl;

        public FlickerRGBVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        public void SetRegisterControl(IRegisterReadWriteEx control)
        {
            _regControl = control;
        }

        // ⭐ 你要的重點：SetModel
        public void SetModel(FlickerRGBModel model)
        {
            _model = model;

            if (_model == null)
            {
                // 沒有 model 時，把 UI 清空成 0
                RPage1 = GPage1 = BPage1 =
                RPage2 = GPage2 = BPage2 =
                RPage3 = GPage3 = BPage3 =
                RPage4 = GPage4 = BPage4 = 0;
                return;
            }

            // Model → VM
            RPage1 = _model.RPage1;
            GPage1 = _model.GPage1;
            BPage1 = _model.BPage1;

            RPage2 = _model.RPage2;
            GPage2 = _model.GPage2;
            BPage2 = _model.BPage2;

            RPage3 = _model.RPage3;
            GPage3 = _model.GPage3;
            BPage3 = _model.BPage3;

            RPage4 = _model.RPage4;
            GPage4 = _model.GPage4;
            BPage4 = _model.BPage4;
        }

        // ===== Page1 =====
        public int RPage1
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.RPage1 = value;   // UI → Model
            }
        }

        public int GPage1
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.GPage1 = value;
            }
        }

        public int BPage1
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.BPage1 = value;
            }
        }

        // ===== Page2 =====
        public int RPage2
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.RPage2 = value;
            }
        }

        public int GPage2
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.GPage2 = value;
            }
        }

        public int BPage2
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.BPage2 = value;
            }
        }

        // ===== Page3 =====
        public int RPage3
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.RPage3 = value;
            }
        }

        public int GPage3
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.GPage3 = value;
            }
        }

        public int BPage3
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.BPage3 = value;
            }
        }

        // ===== Page4 =====
        public int RPage4
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.RPage4 = value;
            }
        }

        public int GPage4
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.GPage4 = value;
            }
        }

        public int BPage4
        {
            get { return GetValue<int>(); }
            set
            {
                if (!SetValue(value)) return;
                if (_model != null) _model.BPage4 = value;
            }
        }

        // ===== 寫入暫存器 =====
        public IRelayCommand ApplyCommand { get; }

        private void ApplyWrite()
        {
            if (_model == null || _regControl == null)
                return;

            // Page1
            if (!string.IsNullOrEmpty(_model.RPage1Reg))
                _regControl.SetRegisterByName(_model.RPage1Reg, (uint)_model.RPage1);
            if (!string.IsNullOrEmpty(_model.GPage1Reg))
                _regControl.SetRegisterByName(_model.GPage1Reg, (uint)_model.GPage1);
            if (!string.IsNullOrEmpty(_model.BPage1Reg))
                _regControl.SetRegisterByName(_model.BPage1Reg, (uint)_model.BPage1);

            // Page2
            if (!string.IsNullOrEmpty(_model.RPage2Reg))
                _regControl.SetRegisterByName(_model.RPage2Reg, (uint)_model.RPage2);
            if (!string.IsNullOrEmpty(_model.GPage2Reg))
                _regControl.SetRegisterByName(_model.GPage2Reg, (uint)_model.GPage2);
            if (!string.IsNullOrEmpty(_model.BPage2Reg))
                _regControl.SetRegisterByName(_model.BPage2Reg, (uint)_model.BPage2);

            // Page3
            if (!string.IsNullOrEmpty(_model.RPage3Reg))
                _regControl.SetRegisterByName(_model.RPage3Reg, (uint)_model.RPage3);
            if (!string.IsNullOrEmpty(_model.GPage3Reg))
                _regControl.SetRegisterByName(_model.GPage3Reg, (uint)_model.GPage3);
            if (!string.IsNullOrEmpty(_model.BPage3Reg))
                _regControl.SetRegisterByName(_model.BPage3Reg, (uint)_model.BPage3);

            // Page4
            if (!string.IsNullOrEmpty(_model.RPage4Reg))
                _regControl.SetRegisterByName(_model.RPage4Reg, (uint)_model.RPage4);
            if (!string.IsNullOrEmpty(_model.GPage4Reg))
                _regControl.SetRegisterByName(_model.GPage4Reg, (uint)_model.GPage4);
            if (!string.IsNullOrEmpty(_model.BPage4Reg))
                _regControl.SetRegisterByName(_model.BPage4Reg, (uint)_model.BPage4);
        }
    }
}
```
