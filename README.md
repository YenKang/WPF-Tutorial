using System.ComponentModel;

namespace YourProjectNamespace.Services // ⚠️ 改成實際命名空間
{
    public class KnobGlowStatus : INotifyPropertyChanged
    {
        private static readonly KnobGlowStatus _instance = new KnobGlowStatus();
        public static KnobGlowStatus Instance => _instance;

        private KnobRole _rightKnobRole = KnobRole.None;
        public KnobRole RightKnobRole
        {
            get => _rightKnobRole;
            set
            {
                if (_rightKnobRole != value)
                {
                    _rightKnobRole = value;
                    OnPropertyChanged(nameof(RightKnobRole));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string propertyName)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}