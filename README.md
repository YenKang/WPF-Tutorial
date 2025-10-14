NovaCID.BistMode/
│
├── ViewModels/
│   ├── BistModeViewModel.cs           ← 主控制邏輯（綁 UI）
│   └── FeatureFlags.cs                ← JSON 開關控制
│
├── Views/
│   └── BistModeView.xaml              ← UI 介面（Pattern list, Gray level, Set 按鈕）
│
├── Services/
│   ├── BistService.cs                 ← 解譯 JSON 指令，寫暫存器
│   ├── RegDisplayControl.cs           ← 你現有的底層 SPI 寫入控制
│   └── ProfileLoader.cs               ← 讀取 JSON 檔、序列化成物件
│
├── Profiles/
│   ├── NT51365.profile.json           ← JSON 設定檔（圖 + 暫存器 + 序列）
│   ├── NT51950.profile.json           ← 未來其他 IC
│   └── NT51920.profile.json
│
├── Assets/
│   └── Patterns/
│       ├── NT51365/
│       │   ├── P0_Black.png
│       │   ├── P1_White.png
│       │   ├── P2_Red.png
│       │   └── ...
│       ├── NT51950/
│       └── NT51920/
│
└── App.xaml.cs
