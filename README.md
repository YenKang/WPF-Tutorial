feat(climate-left): 完成左側溫度與風量控制切換與視覺高亮邏輯

- 新增 LeftControlTarget 狀態（Temp / Fan），集中管理控制目標
- 實作 SelectLeftTempCmd / SelectLeftFanCmd 切換控制區域
- 建立 IsLeftTempControlled / IsLeftFanControlled 判斷屬性，配合外框顯示狀態
- 調整左溫度區塊 UI，支援點擊切換控制權並顯示高亮框
- 重構左風量區塊 UI：
  - 外層 Border 套用 DataTrigger 顯示控制狀態
  - 移除每一條 FanBar 的個別 Border
  - 點擊任一風量條切換控制權
- 整合左 Knob 旋轉事件，依據目前控制區調整 LeftTemperature 或 LeftFanLevel 數值

✅ 操作測試通過：能正確點擊切換控制目標並旋轉控制對應數值