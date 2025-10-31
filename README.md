public void LoadFrom(object json)
{
    Reset();

    var jsonObj = json as JObject;
    if (jsonObj == null) return;

    _isHydrating = true;
    try
    {
        Title = (string)jsonObj["title"] ?? Title;
        _target = (string)jsonObj["target"] ?? _target;

        // 1️⃣ 快取優先（有快取就直接還原 UI 並離開）
        if (LineExclamCursorCache.HasMemory)
        {
            ExclamWhiteBG = LineExclamCursorCache.ExclamWhiteBG;
            return;
        }

        // 2️⃣ 沒快取 → 載入 JSON default
        if (!(jsonObj["fields"] is JObject fields)) return;

        ShowLineSel  = fields["lineSel"]["visible"]?.Value<bool>() ?? false;
        ShowExclamBg = fields["exclamBg"]["visible"]?.Value<bool>() ?? false;
        ShowDiagonal = fields["diagonalEn"]["visible"]?.Value<bool>() ?? false;
        ShowCursor   = fields["cursorEn"]["visible"]?.Value<bool>() ?? false;

        // --- Bit Masks ---
        _maskLineSel  = ReadMask(fields["lineSel"],   _maskLineSel);
        _maskExclamBg = ReadMask(fields["exclamBg"],  _maskExclamBg);
        _maskDiagonal = ReadMask(fields["diagonalEn"],_maskDiagonal);
        _maskCursor   = ReadMask(fields["cursorEn"],  _maskCursor);

        // --- Default States ---
        LineSelWhite  = ReadDefault01(fields["lineSel"])    == 1;
        ExclamWhiteBG = ReadDefault01(fields["exclamBg"])   == 1;
        DiagonalEn    = ReadDefault01(fields["diagonalEn"]) == 1;
        CursorEn      = ReadDefault01(fields["cursorEn"])   == 1;

        // 3️⃣ 將第一次 JSON default 狀態也寫入 Cache（避免紅框）
        LineExclamCursorCache.SaveExclam(ExclamWhiteBG);

        // --- 其他游標座標設定照舊 ---
        // ParsePositionField(...HPOS...)
        // ParsePositionField(...VPOS...)
    }
    finally
    {
        _isHydrating = false;
    }
}