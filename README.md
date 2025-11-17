## Step 1：在 class 裡加 min / max / default 欄位

```
private JObject _cfg;

private int _min = 0x000;
private int _max = 0x3FF;
private int _default = 0x3FF;
```

## step2: 調整 GrayLevel_Text 的 setter（只改這個 property）

```
public string GrayLevel_Text
{
    get { return GetValue<string>(); }
    set
    {
        // 先把原始字串存起來，讓 Binding 有變化通知
        SetValue(value);

        if (string.IsNullOrWhiteSpace(value))
            return;

        try
        {
            var s = value.Trim();

            // 允許輸入 "3FF" 或 "0x3FF"
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                s = s.Substring(2);

            int v;
            if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v))
                return;

            // 範圍限制 [min, max]
            if (v < _min) v = _min;
            if (v > _max) v = _max;

            // 寫入內部的 int 值
            GrayLevel_Value = v;

            // 把顯示字串規一成 3 位 HEX（例如 0x01E → "01E"）
            var normalized = v.ToString("X3");

            // 再寫一次，更新顯示（SetValue 不會重新進到 setter 裡）
            SetValue(normalized);
        }
        catch
        {
            // 解析失敗就先忽略，不彈 MessageBox，避免打字時很吵
        }
    }
}
```

## Step3: LoadFrom

```
// 從 JSON 的 grayLevelControl 區塊載入
public void LoadFrom(object jsonConfig)
{
    _cfg = jsonConfig as JObject;
    var root = _cfg;
    if (root == null)
        return;

    // grayLevelControl.grayLevel 區塊
    var grayCtrl = root["grayLevelControl"] as JObject;
    if (grayCtrl == null)
        return;

    var gl = grayCtrl["grayLevel"] as JObject;
    if (gl == null)
        return;

    // 讀取 min / max / default（允許 "0x3FF" 或 "3FF"）
    _min     = ReadHex(gl, "min",     0x000);
    _max     = ReadHex(gl, "max",     0x3FF);
    _default = ReadHex(gl, "default", 0x3FF);

    // 初始化目前值
    GrayLevel_Value = _default;
    GrayLevel_Text  = _default.ToString("X3");

    Debug.WriteLine($"[LoadFrom] GrayLevel range 0x{_min:X3}~0x{_max:X3}, default=0x{_default:X3}");
}
```

## Step4: Execute Write

```
private void ExecuteWrite()
{
    // 先把一個值轉成 10-bit Gray, 並計算 Low/High
    byte srcVal_low  = (byte)(GrayLevel_Value & 0xFF);
    byte srcVal_high = (byte)((GrayLevel_Value >> 8) & 0xFF);

    var writes = _cfg?["writes"] as JArray;
    if (writes == null || writes.Count == 0)
    {
        MessageBox.Show("此圖 json 格式有誤");
        return;
    }

    // 依 JSON 的每筆 write 執行
    for (int i = 0; i < writes.Count; i++)
    {
        JObject w   = (JObject)writes[i];
        string mode = (string)w["mode"] ?? "write8";
        string target = (string)w["target"];
        if (string.IsNullOrEmpty(target)) continue;

        if (mode == "write8")
        {
            RegMap.Write8(target, srcVal_low);
        }
        else if (mode == "rmw")
        {
            int mask = ParseInt((string)w["mask"]) ?? 0x03;
            int shift = ParseInt((string)w["shift"]) ?? 0;

            byte cur = RegMap.Read8(target);
            byte next = (byte)((cur & (~mask & 0xFF)) |
                              (((srcVal_high << shift) & mask) & 0xFF));

            RegMap.Write8(target, next);
        }
    }
}
```