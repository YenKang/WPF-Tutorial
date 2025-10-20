using System;
using System.Globalization;
using Newtonsoft.Json.Linq;

public void LoadFrom(object json)
{
    // 先清空成預設（你原本的 Reset()）
    Reset();

    var jo = json as JObject;
    if (jo == null) return;

    // 讀 Title 與 Target
    var t = jo["title"];
    if (t != null) Title = (string)t;

    var tgt = jo["target"];
    if (tgt != null) _target = (string)tgt;

    // 讀 fields
    var fields = jo["fields"] as JObject;
    if (fields == null) return;

    // ---- 可見性旗標（四個 bit 對應 UI 是否顯示）----
    ShowLineSel  = fields["lineSel"]    != null && fields["lineSel"]["visible"]    != null && fields["lineSel"]["visible"].Value<bool>();
    ShowExclamBg = fields["exclamBg"]   != null && fields["exclamBg"]["visible"]   != null && fields["exclamBg"]["visible"].Value<bool>();
    ShowDiagonal = fields["diagonalEn"] != null && fields["diagonalEn"]["visible"] != null && fields["diagonalEn"]["visible"].Value<bool>();
    ShowCursor   = fields["cursorEn"]   != null && fields["cursorEn"]["visible"]   != null && fields["cursorEn"]["visible"].Value<bool>();

    // ---- bit mask（缺省值用預設）與 default ----
    _mLine   = ReadMask(fields["lineSel"],    0x20); // B5
    _mExclam = ReadMask(fields["exclamBg"],   0x10); // B4
    _mDiag   = ReadMask(fields["diagonalEn"], 0x08); // B3
    _mCursor = ReadMask(fields["cursorEn"],   0x04); // B2

    LineSelWhite  = ReadDefault01(fields["lineSel"])    == 1;
    ExclamWhiteBG = ReadDefault01(fields["exclamBg"])   == 1;
    DiagonalEn    = ReadDefault01(fields["diagonalEn"]) == 1;
    CursorEn      = ReadDefault01(fields["cursorEn"])   == 1;

    // ---- Cursor Position（可選）----
    bool show; int def, min, max; string low, high; byte mask; int shift;

    // HPos：高位 mask 缺省給 0x1F（對應 12..8 五個 bit）
    if (fields["hPos"] != null)
    {
        ParsePositionField(fields["hPos"],
            out show, out def, out min, out max,
            out low, out high, out mask, out shift,
            0x1F);

        ShowHPos     = show;
        HPos         = def;
        _hMin        = min;
        _hMax        = max;
        _hLowTarget  = low;
        _hHighTarget = high;
        _hMask       = mask;
        _hShift      = shift;
    }

    // VPos：高位 mask 缺省給 0x0F（對應 11..8 四個 bit）
    if (fields["vPos"] != null)
    {
        ParsePositionField(fields["vPos"],
            out show, out def, out min, out max,
            out low, out high, out mask, out shift,
            0x0F);

        ShowVPos     = show;
        VPos         = def;
        _vMin        = min;
        _vMax        = max;
        _vLowTarget  = low;
        _vHighTarget = high;
        _vMask       = mask;
        _vShift      = shift;
    }

    // 讓綁定到 IsHPosVisible / IsVPosVisible 的元素立刻刷新
    OnPropertyChanged("IsHPosVisible");
    OnPropertyChanged("IsVPosVisible");
}