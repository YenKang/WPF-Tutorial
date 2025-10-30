// 依 datasheet 調整這裡
private static (string reg, byte mask, int shift) GetAxisSpec(string axis)
{
    const string REG = "BIST_GrayColor_VH_Reverse";
    return (axis == "V")
        ? (REG, (byte)0x20, 5)  // V: bit5
        : (REG, (byte)0x10, 4); // H: bit4
}

＝＝＝＝＝＝

public void LoadFrom(object jsonCfg)
{
    _cfg = jsonCfg as JObject;
    if (_cfg == null) return;

    // 讀 axis (預設 H)
    var axisTok = _cfg["axis"];
    var axis = axisTok != null ? axisTok.ToString().Trim().ToUpperInvariant() : "H";
    Axis = (axis == "V") ? "V" : "H";

    // 讀 default（0/1，僅供初始顯示）
    int def = (int?)_cfg["default"] ?? 0;
    Enable = (def != 0);

    // 關鍵：載入後立刻以暫存器實際值覆蓋 UI
    TryRefreshFromRegister();
}