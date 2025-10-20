public void LoadFrom(object json)
{
    Reset();
    if (!(json is JObject jsonObj)) return;

    Title = (string)jsonObj["title"] ?? Title;
    _target = (string)jsonObj["target"] ?? _target;

    var fields = jsonObj["fields"] as JObject;
    if (fields == null) return;

    // === 解析可見性 ===
    ShowLineSel   = fields["lineSel"]?["visible"]?.Value<bool>() ?? false;
    ShowExclamBg  = fields["exclamBg"]?["visible"]?.Value<bool>() ?? false;
    ShowDiagonal  = fields["diagonalEn"]?["visible"]?.Value<bool>() ?? false;
    ShowCursor    = fields["cursorEn"]?["visible"]?.Value<bool>() ?? false;

    // === 解析 bit mask ===
    _lineSelMask  = ReadMask(fields["lineSel"],   _lineSelMask);
    _exclamMask   = ReadMask(fields["exclamBg"],  _exclamMask);
    _diagMask     = ReadMask(fields["diagonalEn"],_diagMask);
    _cursorMask   = ReadMask(fields["cursorEn"],  _cursorMask);

    // === 預設值 ===
    LineSelWhite  = ReadDefault01(fields["lineSel"])    == 1;
    ExclamWhiteBG = ReadDefault01(fields["exclamBg"])   == 1;
    DiagonalEn    = ReadDefault01(fields["diagonalEn"]) == 1;
    CursorEn      = ReadDefault01(fields["cursorEn"])   == 1;

    // === HPos/VPos ===
    if (fields["hPos"] != null)
    {
        var hp = fields["hPos"];
        ShowHPos = hp["visible"]?.Value<bool>() ?? false;
        HPos = ReadIntDefault(hp);
        _hMin = hp["min"]?.Value<int>() ?? 0;
        _hMax = hp["max"]?.Value<int>() ?? 0;
        _hLowTarget = (string)hp["low"];
        _hHighTarget = (string)hp["high"];
        _hMask = ReadMask(hp, 0x1F);
    }

    if (fields["vPos"] != null)
    {
        var vp = fields["vPos"];
        ShowVPos = vp["visible"]?.Value<bool>() ?? false;
        VPos = ReadIntDefault(vp);
        _vMin = vp["min"]?.Value<int>() ?? 0;
        _vMax = vp["max"]?.Value<int>() ?? 0;
        _vLowTarget = (string)vp["low"];
        _vHighTarget = (string)vp["high"];
        _vMask = ReadMask(vp, 0x0F);
    }
}