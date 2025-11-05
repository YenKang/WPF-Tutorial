feat(osd-icon-flash-map): 完成 OSD ↔ Icon 動態連動機制

• 新增 IconFlashMapWindow.xaml，具備四欄表格（OSD / Icon / 縮圖 / FlashStartAddr）
• 採 MVVM 架構（View、Model、ViewModel），不分 Core/UI 專案
• 實作 OsdModel 衍生屬性（ThumbnailKey、FlashStartAddrHex），綁定 SelectedIcon 變更自動更新
• XAML 綁定改為顯示/編輯分離，確保換 ICON 後縮圖與 Flash Address 即時刷新
• 新增 IC Select（Primary / Secondary / Teritary）與假資料載入邏輯
• 改善程式可讀性（展開 ?. / ?? 為明確 if/else）
• 統一 RaisePropertyChanged 寫法，符合公司規範