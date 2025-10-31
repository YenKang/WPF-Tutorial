// ---------- FCNT2 ----------
var f2 = autoRunControl["fcnt2"] as JObject;
_fc2Default = ParseHex((string)f2?["default"], 0x01E);  // 預設 0x01E

if (f2?["targets"] is JObject t2)
{
    _fc2LowTarget  = (string)t2["low"];
    _fc2HighTarget = (string)t2["high"];
    _fc2Mask       = ParseHexByte((string)t2["mask"], 0x0C);
    _fc2Shift      = (int?)t2["shift"] ?? 10;
}

// 將值展開到 UI：先看快取，沒有就吃 JSON default
if (AutoRunCache.HasMemory)
{
    UnpackFcnt2ToUI(AutoRunCache.Fcnt2);
}
else
{
    UnpackFcnt2ToUI(_fc2Default);
}