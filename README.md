using System;
using System.Globalization;
using Newtonsoft.Json.Linq;

private static void ParsePositionField(
    JToken field,
    out bool visible, out int def,
    out int min, out int max,
    out string lowTarget, out string highTarget,
    out byte highMask, out int highShift,
    byte defaultMaskIfMissing)
{
    // 預設輸出
    visible    = false;
    def        = 0;
    min        = 0;
    max        = 0;
    lowTarget  = null;
    highTarget = null;
    highMask   = 0;
    highShift  = 0;

    var fo = field as JObject;
    if (fo == null) return;

    // 基本欄位
    var visTok = fo["visible"];
    visible = visTok != null && visTok.Type != JTokenType.Null && visTok.Value<bool>();

    min = (int?)(fo["min"]) ?? 0;
    max = (int?)(fo["max"]) ?? 0;
    def = (int?)(fo["default"]) ?? 0;
    if (def < min) def = min;
    else if (def > max) def = max;

    // write.low / write.high
    var write = fo["write"] as JObject;
    if (write == null) return;

    var low = write["low"] as JObject;
    if (low != null)
        lowTarget = (string)low["target"];

    var high = write["high"] as JObject;
    if (high != null)
    {
        highTarget = (string)high["target"];

        // 解析 mask（十六進位字串或缺省）
        string maskStr = (string)high["mask"];
        if (string.IsNullOrWhiteSpace(maskStr))
        {
            highMask = defaultMaskIfMissing;
        }
        else
        {
            if (maskStr.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
                maskStr = maskStr.Substring(2);

            byte parsed;
            if (byte.TryParse(maskStr, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out parsed))
                highMask = parsed;
            else
                highMask = defaultMaskIfMissing;
        }

        // 解析 shift（可省略；省略時由 mask 推算最低位）
        int? shiftNullable = (int?)high["shift"];
        if (shiftNullable.HasValue)
        {
            highShift = shiftNullable.Value;
        }
        else
        {
            int s = 0;
            byte m = highMask;
            while (s < 8 && ((m >> s) & 1) == 0) s++;
            highShift = s;
        }
    }
}