🔧 重構 Knob 事件處理流程與狀態比對邏輯

- 完整實作 KnobStatus / KnobEvent / KnobEventProcessor
- 加入 Rotate / Press 行為定義與比對邏輯
- 建立 KnobEventRouter 分派至 IKnobHandler
- ClimatePageViewModel 成功接收 Knob 操作事件
- 加入 Debug Log 與溫度數值調整邏輯
- 整合 NovaCIDViewModel 與 OnRawKnobDataReceived()