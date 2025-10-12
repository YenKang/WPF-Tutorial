NovaCID（現有舊架構）
├─ NovaCIDViewModel.cs  ← 集中式控制核心
│  ├─ 管理頁面切換、Knob 顯示/座標、讀取 Setting.xml
│  ├─ 包含：
│  │  ├─ HomeViewModel
│  │  ├─ DriverPageViewModel
│  │  ├─ MusicPageViewModel
│  │  ├─ CurrentPage / SelectedMode
│  │  ├─ Knob 相關 (HasKnob, DknobX/Y)
│  │  └─ 雙固定旋鈕位置 (Left/Right Knob)
│  └─ 編譯期綁定：
│     ├─ SwitchPage() 以 switch-case 決定顯示頁
│     ├─ IconDock 綁定固定頁面
│     └─ XAML 內 DataTemplate 寫死綁定關係
│
├─ Views
│  ├─ NovaCIDView.xaml  (主畫面)
│  ├─ HomeView.xaml
│  ├─ DriverPageView.xaml
│  └─ MusicPageView.xaml
│
├─ ViewModels
│  ├─ HomeViewModel.cs
│  ├─ DriverPageViewModel.cs
│  └─ MusicPageViewModel.cs
│
├─ IconDock（中間底部）
│  ├─ IconItem：Home / Driver / Music
│  └─ 點擊 icon → 切換 CurrentPage
│
└─ Setting.xml
   ├─ HasKnob：是否顯示旋鈕
   ├─ DynamicMode：是否為單顆動態旋鈕
   ├─ DknobX / DknobY：動態座標
   └─ Left/Right 固定座標設定
