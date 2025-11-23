using System;
using System.Collections.ObjectModel;
using System.Diagnostics;
using Utility.MVVM;
using Utility.MVVM.Command;
using BistMode.Models;

namespace BistMode.ViewModels
{
    public sealed class AutoRunVM : ViewModelBase
    {
        private AutoRunModel _model;
        private IRegisterControl _regControl;

        public AutoRunVM()
        {
            ApplyCommand = CommandFactory.CreateCommand(ApplyWrite);
        }

        public void SetRegisterControl(IRegisterControl control)
        {
            _regControl = control;
        }

        // -------------------------------------------------------------
        // UI Bindings
        // -------------------------------------------------------------
        public string Title
        {
            get => GetValue<string>();
            set => SetValue(value);
        }

        public int Total
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;

                if (_model != null)
                    _model.Total = value;

                Debug.WriteLine($"[AutoRunVM] Total = {value}");
            }
        }

        // 每一筆 OrderSlot = (Index, Value, RegName)
        public ObservableCollection<OrderSlot> Orders { get; private set; }
            = new ObservableCollection<OrderSlot>();

        public int Fcnt1
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;

                if (_model != null)
                    _model.Fcnt1 = value;

                Debug.WriteLine($"[AutoRunVM] Fcnt1 = {value:X}");
            }
        }

        public int Fcnt2
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;

                if (_model != null)
                    _model.Fcnt2 = value;

                Debug.WriteLine($"[AutoRunVM] Fcnt2 = {value:X}");
            }
        }

        public IRelayCommand ApplyCommand { get; }

        // -------------------------------------------------------------
        // STEP 3：SetModel（切圖時呼叫）
        // -------------------------------------------------------------
        public void SetModel(AutoRunModel model)
        {
            _model = model;

            Orders.Clear();

            if (model == null)
            {
                Debug.WriteLine("[AutoRunVM] SetModel(null)");
                return;
            }

            Debug.WriteLine("=== [AutoRunVM] SetModel START ===");

            Title = model.Title;

            // Total
            Total = model.Total;
            Debug.WriteLine($"  Total = {Total}");

            // Pattern Orders
            for (int i = 0; i < model.OrderValues.Count; i++)
            {
                var slot = new OrderSlot
                {
                    Index = i + 1,
                    Value = model.OrderValues[i],
                    RegName = model.OrderRegs[i]
                };

                // UI 改動 → 自動更新 Model
                slot.PropertyChanged += (s, e) =>
                {
                    if (_model != null)
                    {
                        _model.OrderValues[i] = slot.Value;
                    }
                };

                Orders.Add(slot);

                Debug.WriteLine($"  Order[{slot.Index}] = {slot.Value}, Reg={slot.RegName}");
            }

            // fcnt1 & fcnt2
            Fcnt1 = model.Fcnt1;
            Fcnt2 = model.Fcnt2;

            Debug.WriteLine($"  Fcnt1 = {Fcnt1:X}");
            Debug.WriteLine($"  Fcnt2 = {Fcnt2:X}");

            Debug.WriteLine("=== [AutoRunVM] SetModel END ===");
        }

        // -------------------------------------------------------------
        // STEP 4：ApplyWrite（寫暫存器）
        // -------------------------------------------------------------
        private void ApplyWrite()
        {
            if (_model == null || _regControl == null)
                return;

            Debug.WriteLine("=== [AutoRunVM] ApplyWrite START ===");

            // Write Total
            if (!string.IsNullOrEmpty(_model.TotalReg))
            {
                Debug.WriteLine($"  Write { _model.TotalReg } = {Total}");
                _regControl.SetRegisterByName(_model.TotalReg, (uint)Total);
            }

            // Write each Order
            foreach (var slot in Orders)
            {
                Debug.WriteLine($"  Write {slot.RegName} = {slot.Value}");
                _regControl.SetRegisterByName(slot.RegName, (uint)slot.Value);
            }

            // fcnt1
            if (!string.IsNullOrEmpty(_model.Fcnt1Reg))
            {
                Debug.WriteLine($"  Write { _model.Fcnt1Reg } = {Fcnt1:X}");
                _regControl.SetRegisterByName(_model.Fcnt1Reg, (uint)Fcnt1);
            }

            // fcnt2
            if (!string.IsNullOrEmpty(_model.Fcnt2Reg))
            {
                Debug.WriteLine($"  Write { _model.Fcnt2Reg } = {Fcnt2:X}");
                _regControl.SetRegisterByName(_model.Fcnt2Reg, (uint)Fcnt2);
            }

            Debug.WriteLine("=== [AutoRunVM] ApplyWrite END ===");
        }
    }

    // -------------------------------------------------------------
    // OrderSlot：不需要 Converter，只綁 Model
    // -------------------------------------------------------------
    public sealed class OrderSlot : ViewModelBase
    {
        public int Index
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        public int Value
        {
            get => GetValue<int>();
            set => SetValue(value);
        }

        public string RegName
        {
            get => GetValue<string>();
            set => SetValue(value);
        }
    }
}
