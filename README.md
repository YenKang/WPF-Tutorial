private bool _isHydrating; // 防止載入時觸發不必要的動作（若你需要）

public void LoadFrom(object json)
{
    Reset();

    var jsonObj = json as JObject;
    if (jsonObj == null) return;

    Title  = (string)jsonObj["title"] ?? Title;
    _target = (string)jsonObj["target"] ?? _target;

    // ① 快取優先（有就直接還原 UI & 結束）
    if (LineExclamCursorCache.HasMemory)
    {
        _isHydrating = true;
        try
        {
            ExclamWhiteBG = LineExclamCursorCache.ExclamWhiteBG;
            // 若你之後還有 Cursor 位置、Line/Diag 類似屬性，也都在這一段還原
        }
        finally
        {
            _isHydrating = false;
        }
        return;
    }

    // ② 沒快取 → 解析 JSON，設定預設值
    if (!(jsonObj["fields"] is JObject fields)) return;

    // 下面維持你原本的解析流程（略）：
    // - ReadMask / ReadDefault01 / ParsePositionField ...
    // - 預設值區：把 ExclamWhiteBG 的 default 設起來
    var exTok = fields["exclamBg"] as JObject;
    if (exTok != null)
    {
        // 你的 default 讀法已經存在：ReadDefault01(...)
        var def = ReadDefault01(exTok); // 1=White, 0=Black
        ExclamWhiteBG = (def != 0);

        // 也順手把 default 存進快取，讓第二次切回來不紅框
        LineExclamCursorCache.SaveExclam(ExclamWhiteBG);
    }

    // 其他欄位（lineSel、diagonal、cursor、HPOS/VPOS 等）照你現有程式處理
}