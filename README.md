# ✅ NovaCID 專案 – 今日進度總結（06/26）

## 🧱 基礎架構完成
- [x] 明確定義 `Rotate` 與 `Press` 行為觸發條件
- [x] 撰寫完整行為規格文件：
  - `RotateBehavior.md`
  - `PressBehavior.md`
  - `KnobSpec.md`

---

## 🔄 Knob 模組核心實作

### ✅ `KnobEvent.cs`
- 定義 `KnobEventType`, `KnobRole`
- 提供工廠方法：
  - `CreateRotate()`
  - `CreatePress()`

### ✅ `KnobStatus.cs`
- 記錄旋鈕狀態：
  - `IsTouched`, `Counter`, `IsPressed`
  - 以及對應的 `PreviousXXX` 值
- 提供 `.Clone()` 方法
- 整合 `KnobIdHelper`，將 `"Knob_0"` 字串轉為 `KnobId.Left` 等 enum 型別

### ✅ `KnobEventProcessor.cs`
- 處理 raw knob data 字串並逐行解析
- 與 `KnobStatusMap` 比對前後狀態
- 正確觸發事件：
  - 旋轉：`delta ≠ 0` 且持續觸碰
  - 按壓：`IsPressed` 從 false → true
- 封裝成 `ProcessEvent(old, current)` 提升可讀性
- 加入 Debug log（含 KnobId 與事件類型）

---

## 🔀 Knob 事件轉發

### ✅ `IKnobEventRouter.cs`
- 定義 `Route(KnobEvent e)` 方法，負責事件轉派

### ✅ `KnobEventRouter.cs`
- 以 `Func<object>` 動態取得目前畫面 ViewModel
- 若實作 `IKnobHandler`，根據 `KnobEventType` 與 `Role` 呼叫對應事件方法：
  - `OnDriverKnobRotated(e)`
  - `OnPassengerKnobRotated(e)`
  - `OnDriverKnobPressed(e)`
  - `OnPassengerKnobPressed(e)`

---

## 📘 `NovaCIDViewModel.cs` 整合
- 新增 `_router`, `_processor` 欄位與建構式注入
- 新增 `OnRawKnobDataReceived(string)` 作為主入口
- 整合 `_currentPageType` 狀態與頁面切換邏輯
- 目前選定 `ClimatePage` 為測試場景

---

## 🧪 測試與除錯紀錄

- [x] 修正 `Role` 解析大小寫錯誤導致事件無法觸發
- [x] 追蹤 `Counter = 0` 問題，解決 clone / status 記錄失敗問題
- [x] 驗證解析格式與狀態記憶是否正確更新
- [x] 加入 Debug log 印出事件內容與狀態差異
- [x] 討論 counter overflow（255 → 0）行為並準備後續處理

---

## 🔢 數值與邏輯處理設計

- ✅ 溫度調整控制：
  - 範圍：16 ~ 28°C
  - 控制靈敏度：`delta / 10.0`
  - 邊界控制：不允許超出上下限

---

## 📌 總結

🎯 成功建構並貫通 **Knob 處理流程全線：**