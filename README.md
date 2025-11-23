using System;
using System.Collections.ObjectModel;
using System.Diagnostics;
using BistMode.Models;
using Utility.MVVM.Command;

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

        public void SetRegisterControl(IRegisterControl ctrl)
        {
            _regControl = ctrl;
        }

        // --------------------------
        // Public VM Properties
        // --------------------------
        public int Total
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                {
                    _model.Total = value;
                }
                Debug.WriteLine($"[AutoRunVM] Total Change: {value}");
            }
        }

        public int Fcnt1
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                {
                    _model.Fcnt1 = value;
                }
                Debug.WriteLine($"[AutoRunVM] Fcnt1 Change: {value}");
            }
        }

        public int Fcnt2
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;
                if (_model != null)
                {
                    _model.Fcnt2 = value;
                }
                Debug.WriteLine($"[AutoRunVM] Fcnt2 Change: {value}");
            }
        }

        public ObservableCollection<OrderSlot> Orders { get; }
            = new ObservableCollection<OrderSlot>();

        public IRelayCommand ApplyCommand { get; }

        // --------------------------
        // SetModel
        // --------------------------
        public void SetModel(AutoRunModel model)
        {
            _model = model;
            if (model == null) return;

            Debug.WriteLine("=== [AutoRunVM] SetModel ===");

            // sync Total / Fcnts
            Total = model.Total;
            Fcnt1 = model.Fcnt1;
            Fcnt2 = model.Fcnt2;

            // Build Slots
            Orders.Clear();

            for (int i = 0; i < model.OrderValues.Count; i++)
            {
                var slot = new OrderSlot(
                    model,
                    i,                      // 0-based index
                    model.OrderValues[i]     // default value
                );

                Orders.Add(slot);

                Debug.WriteLine($"[AutoRunVM] Slot Added: Index={i}, Val={model.OrderValues[i]}");
            }

            Debug.WriteLine("=== [AutoRunVM] SetModel Done ===");
        }

        // --------------------------
        // Write Registers
        // --------------------------
        private void ApplyWrite()
        {
            if (_model == null || _regControl == null)
                return;

            Debug.WriteLine("=== [AutoRun Apply] ===");

            // total
            if (!string.IsNullOrEmpty(_model.TotalReg))
            {
                _regControl.SetRegisterByName(_model.TotalReg, (uint)_model.Total);
                Debug.WriteLine($"[AutoRun] Write Total: {_model.TotalReg} = {_model.Total}");
            }

            // pattern orders
            for (int i = 0; i < _model.OrderRegs.Count; i++)
            {
                string reg = _model.OrderRegs[i];
                int val  = _model.OrderValues[i];

                _regControl.SetRegisterByName(reg, (uint)val);

                Debug.WriteLine($"[AutoRun] Write Order: {reg} = {val}");
            }

            // fcnt1
            if (!string.IsNullOrEmpty(_model.Fcnt1Reg))
            {
                _regControl.SetRegisterByName(_model.Fcnt1Reg, (uint)_model.Fcnt1);
                Debug.WriteLine($"[AutoRun] Write Fcnt1: {_model.Fcnt1Reg} = {_model.Fcnt1}");
            }

            // fcnt2
            if (!string.IsNullOrEmpty(_model.Fcnt2Reg))
            {
                _regControl.SetRegisterByName(_model.Fcnt2Reg, (uint)_model.Fcnt2);
                Debug.WriteLine($"[AutoRun] Write Fcnt2: {_model.Fcnt2Reg} = {_model.Fcnt2}");
            }

            Debug.WriteLine("=== [AutoRun Apply Done] ===");
        }
    }

    // ==========================================
    // OrderSlot：單一欄位（Value ↔ Model）
    // ==========================================
    public sealed class OrderSlot : ViewModelBase
    {
        private readonly AutoRunModel _model;

        public int Index { get; }

        public OrderSlot(AutoRunModel model, int index, int value)
        {
            _model = model;
            Index  = index;
            SetValue(value);
        }

        public int Value
        {
            get => GetValue<int>();
            set
            {
                if (!SetValue(value)) return;

                _model.OrderValues[Index] = value;

                Debug.WriteLine($"[AutoRun][Slot] Index={Index} => Value={value}");
            }
        }
    }
}
