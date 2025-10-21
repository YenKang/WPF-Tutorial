public void LoadFrom(JObject autoRunControl)
{
    if (autoRunControl == null) return;

    Title = (string)autoRunControl["title"] ?? "Auto Run";

    // === 1️⃣ 讀取 total ===
    var totalObj = autoRunControl["total"] as JObject;
    _totalTarget = (string)totalObj?["target"];
    var totalMin = totalObj?["min"]?.Value<int>() ?? 1;
    var totalMax = totalObj?["max"]?.Value<int>() ?? 22;
    var totalDef = totalObj?["default"]?.Value<int>() ?? 5;
    Total = Math.Max(totalMin, Math.Min(totalMax, totalDef));

    // === 2️⃣ 讀取 orders ===
    var ordersObj = autoRunControl["orders"] as JObject;
    _orderTargets = ordersObj?["targets"]?.ToObject<string[]>() ?? new string[0];
    var defaults  = ordersObj?["defaults"]?.ToObject<int[]>() ?? Array.Empty<int>();

    Orders.Clear();
    for (int i = 0; i < Total; i++)
    {
        Orders.Add(new OrderSlot
        {
            DisplayNo     = i + 1,
            SelectedIndex = (i < defaults.Length) ? defaults[i] : 0
        });
    }

    // === ✅ 3️⃣ 讀取 options (重點) ===
    var optArray = autoRunControl["options"] as JArray;
    if (optArray != null && optArray.Count > 0)
    {
        PatternOptions.Clear();
        foreach (var t in optArray.OfType<JObject>())
        {
            var idx  = t["index"]?.Value<int>() ?? 0;
            var name = (string)t["name"] ?? idx.ToString();
            PatternOptions.Add(new PatternOption { Index = idx, Display = name });
        }
    }

    // === 4️⃣ 讀取 trigger ===
    var trig = autoRunControl["trigger"] as JObject;
    _triggerTarget = (string)trig?["target"];
    _triggerValue  = ParseHexByte((string)trig?["value"], 0x3F);
}