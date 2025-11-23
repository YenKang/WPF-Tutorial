public sealed class AutoRunVM : ViewModelBase
{
    // AutoRunVM 成員...

    // -----------------------------
    // Slot class 放在 AutoRunVM 下面
    // -----------------------------
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
