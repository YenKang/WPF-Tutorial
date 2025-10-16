Profile JSON ──▶ PatternNode ──▶ PatternItem ──▶ ViewModel + XAML
   │                   │                  │
   │                   │                  ├───▶ IsGrayLevelVisible → 控制顯示 GroupBox
   │                   │                  ├───▶ GrayLevelControl → 傳給 GrayLevelVM.LoadFrom()
   │                   │                  └───▶ RegControlType → 決定支援哪些控制模組