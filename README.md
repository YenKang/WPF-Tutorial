private static void ParsePositionField(
    JToken field,
    out bool visible, out int def,
    out int min, out int max,
    out string lowTarget, out string highTarget,
    out byte highMask, out int highShift,
    byte defaultMaskIfMissing)
{
    visible    = field?["visible"]?.Value<bool>() ?? false;
    def        = field?["default"]?.Value<int>() ?? 0;
    min        = field?["min"]?.Value<int>() ?? 0;
    max        = field?["max"]?.Value<int>() ?? 0;
    def        = Math.Max(min, Math.Min(max, def));

    lowTarget  = null;
    highTarget = null;
    highMask   = 0;
    highShift  = 0;

    var write = field?["write"] as JObject;
    if (write != null)
    {
        var low  = write["low"]  as JObject;
        var high = write["high"] as JObject;

        lowTarget  = (string)low?["target"];

        if (high != null)
        {
            highTarget = (string)high["target"];
            var maskStr = (string)high["mask"];
            highMask  = string.IsNullOrWhiteSpace(maskStr) ? defaultMaskIfMissing
                       : Convert.ToByte(maskStr.Replace("0x",""), 16);
            // 允許 JSON 不填 shift：自動從 mask 推出起始位
            if (high["shift"] != null) highShift = high["shift"]!.Value<int>();
            else
            {
                int s = 0; byte m = highMask;
                while (s < 8 && ((m >> s) & 1) == 0) s++;
                highShift = s;
            }
        }
    }
}