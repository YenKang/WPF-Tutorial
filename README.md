# WPF

# 🧭 NovaCID 專案 - Knob 操作規格書

_Last updated: 2025/06/26_

本文件定義 NovaCID 專案中，Knob 旋鈕（Knob_0 與 Knob_1）之 Rotate 與 Press 行為的完整紅線規範、狀態流程與對應實作元件。

---

## 🎛 Knob 命名與角色定義

| Knob 名稱 | 實體位置   | 對應角色     |
|-----------|------------|--------------|
| `Knob_0`  | 左旋鈕     | Driver（主駕）|
| `Knob_1`  | 右旋鈕     | Passenger（副駕）|

每筆 raw string 會包含以下欄位：
```
Knob_0,Role=Driver,Touch=True,Counter=13,Press=False
Knob_1,Role=Passenger,Touch=True,Counter=4,Press=True
```
---

## 🔁 Knob Rotate 行為定義

### ✅ 觸發條件（紅線）

- `Touch == true`
- `Counter` 數值變化（相較上次）
- `Delta ≠ 0`（有旋轉方向）

📌 若 `Touch == false`，即使 Counter 變化也不觸發 Rotate

---

### 🔄 Rotate 狀態流程

---

## 🔁 Knob Rotate 行為定義

### ✅ 觸發條件（紅線）

- `Touch == true`
- `Counter` 數值變化（相較上次）
- `Delta ≠ 0`（有旋轉方向）

📌 若 `Touch == false`，即使 Counter 變化也不觸發 Rotate

---

### 🔄 Rotate 狀態流程


---

### 🧪 Rotate 範例

```text
Knob_0,Touch=False,Counter=10
Knob_0,Touch=True,Counter=10
Knob_0,Touch=True,Counter=11   ✅ Rotate +1
Knob_0,Touch=True,Counter=13   ✅ Rotate +2
Knob_0,Touch=False,Counter=13  ← 結束




---

### 🧪 Rotate 範例

```text
Knob_0,Touch=False,Counter=10
Knob_0,Touch=True,Counter=10
Knob_0,Touch=True,Counter=11   ✅ Rotate +1
Knob_0,Touch=True,Counter=13   ✅ Rotate +2
Knob_0,Touch=False,Counter=13  ← 結束

---
## Press State Flow

[Idle]
   ↓ Touch=True
[被觸控中]
   ↓ Press: false → true
✅ 觸發 Press 行為
   ↓ Touch=False & Press=False
[Idle]

## Press Example

Knob_1,Touch=False,Press=False
Knob_1,Touch=True,Press=False
Knob_1,Touch=True,Press=True     ✅ Press Triggered!
Knob_1,Touch=True,Press=True     ❌ 不再觸發
Knob_1,Touch=False,Press=False   ← Reset 完整結束








