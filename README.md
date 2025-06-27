Knob Raw Data (string)
        ↓
NovaCIDViewModel.OnRawKnobDataReceived(string rawData)
        ↓
KnobEventProcessor.ParseAndProcess(rawData)
        ↓
KnobEventRouter.Route(KnobEvent e)
        ↓
目前頁面 ViewModel (需實作 IKnobHandler)
        ↓
ViewModel 中對應事件方法被呼叫 (如 OnDriverKnobRotated)
        ↓
更新畫面狀態（ZoomLevel / 溫度 / 音樂播放模式等）