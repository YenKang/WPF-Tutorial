NovaCID（主程式）
├─ ModeHost（主控中樞，兼容新舊）
│  ├─ LegacyProvider            ← 舊有頁面提供器（Home、Driver、Music…）
│  ├─ PluginProvider            ← 新的外掛頁掃描器（./Modes/*.dll）
│  ├─ Pages = Legacy ∪ Plugins  ← 最終頁面清單（新舊共存）
│  └─ ViewResolver              ← 動態決定頁面外觀
│     ├─ 1️⃣ ./Overrides/<Id>/View.xaml（外部覆蓋）
│     ├─ 2️⃣ Plugin DLL 內建 View
│     └─ 3️⃣ Legacy DataTemplate（最後回退）
│
├─ UI
│  ├─ IconDock（底部選單，顯示所有頁面）
│  │   └─ ItemsSource = Pages
│  └─ ContentPresenter
│      └─ Content = ViewResolver.Resolve(ActivePage)
│
├─ Legacy Pages（舊頁面仍可正常運作）
│  ├─ HomeViewModel / View
│  ├─ DriverPageViewModel / View
│  └─ MusicPageViewModel / View
│
├─ Modes/         ← 新增「外掛頁面」DLL
│  ├─ MovieMode.dll
│  ├─ ClimateMode.dll
│  └─ ...
│
└─ Overrides/     ← 外部 XAML 改外觀（不重編）
   ├─ MovieMode/View.xaml
   └─ ClimateMode/View.xaml
