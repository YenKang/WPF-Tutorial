### feat(osd-icon-map): 完成七欄 Icon→Image 對應頁面與圖片牆（穩定版）

- 新增 IconToImageMapWindow（七欄 DataGrid）
  Icon# / Image Selection / SRAM / OSD / EN / HPos / VPos
- 採 DataGrid Header 自動對齊；排版整齊。
- IconSlotModel 改用 GetValue/SetValue 格式。
- ImagePickerWindow：
  - 顯示所有圖片、不過濾。
  - 已被其他列使用的圖片有標示，仍可重複選。
  - 本列選中圖片以藍框顯示。
- IconToImageMapWindow.xaml.cs：
  - 建構子確保初始化（IconSlots + Images）。
  - `OpenPicker_Click`：開啟圖片牆，標示使用中項。
  - `ConfirmAndClose`：輸出 `ResultMap (IconIndex→ImageOption)`。
- 加入 OK/Cancel 按鈕，支援 ShowDialog() 模式。

✅ 成功顯示 30 列資料、圖片牆可正常選取。
🚧 待辦：IC Selection / HPos-VPos 驗證 / SRAM 外部設定 / 儲存功能。