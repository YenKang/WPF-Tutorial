// ---- Cursor Position (optional) ----
if (fields?["hPos"] != null)
{
    bool show; int def, min, max; string lowTarget, highTarget; byte mask;
    ParsePositionField(fields["hPos"], out show, out def,
        out min, out max, out lowTarget, out highTarget, out mask, 0x1F);

    ShowHPos      = show;   // 是否顯示 HPos 控件
    HPos          = def;    // 預設值
    _hMin         = min;
    _hMax         = max;
    _hLowTarget   = lowTarget;   // 如 "BIST_CURSOR_HPOS_LO" (0045h)
    _hHighTarget  = highTarget;  // 如 "BIST_CURSOR_HPOS_HI" (0046h)
    _hMask        = mask;        // 高位有效遮罩（HPOS 用 0x1F）
}

if (fields?["vPos"] != null)
{
    bool show; int def, min, max; string lowTarget, highTarget; byte mask;
    ParsePositionField(fields["vPos"], out show, out def,
        out min, out max, out lowTarget, out highTarget, out mask, 0x0F);

    ShowVPos      = show;   // 是否顯示 VPos 控件
    VPos          = def;    // 預設值
    _vMin         = min;
    _vMax         = max;
    _vLowTarget   = lowTarget;   // 如 "BIST_CURSOR_VPOS_LO" (0047h)
    _vHighTarget  = highTarget;  // 如 "BIST_CURSOR_VPOS_HI" (0048h)
    _vMask        = mask;        // 高位有效遮罩（VPOS 用 0x0F）
}