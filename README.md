      🔄 View（UI / XAML 綁定）
              │
        綁定 ZoomLevel
              ↓
─────────────────────────────
  💡 DrivePageViewModel : INotifyPropertyChanged
─────────────────────────────
  ┌────────────┐
  │ DriveState │ ←─📦  Model 屬性
  └────────────┘
       │
       └─ public double ZoomLevel { get; set; } = 1;
