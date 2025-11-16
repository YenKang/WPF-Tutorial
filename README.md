refactor(osd-sram): 重構 SRAM 位址計算機制，改為依據 Image Selected 的順序動態生成

• 新增 SramButton Model + SramButtonList，用於儲存每個 SRAM Slot 對應的圖片資訊
• SRAM 位址計算改為依「已選取圖片的順序」產生 SRAM1、SRAM2、SRAM3…
• 未選圖片（SelectedImage == null）不再佔用 SRAM Slot，僅顯示 "-"
• 移除 ImageOption.SramByteSize，統一使用 _imageSizeMap[key] 取得圖片 Byte Size
• IconSlotModel.SelectedImage 變更時會自動觸發 SRAM 全表重算
• LoadImagesFromFlashButtons 改為建立 Images 清單並同步建立 _imageSizeMap
• 新增 SRAM Mapping Debug Dump，方便驗證地址是否正確
• 清理冗餘程式碼，使圖片來源與 SRAM 計算邏輯更加一致、可維護