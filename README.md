NovaCID（主程式）
├─ ModeHost（一次寫好 Loader）
│  └─ 掃描 ./Modes/*.dll → 建立清單
│
├─ Modes/
│  ├─ HomeMode.dll
│  ├─ DriverMode.dll
│  └─ MovieMode.dll   ← 你新增的外掛頁（丟檔即可出現）
│
└─ Overrides/
   └─ MovieMode/
      └─ View.xaml     ← 想改外觀就放這裡
