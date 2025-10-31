private string[] _orderTargets = InitDefaultOrderTargets();

private static string[] InitDefaultOrderTargets()
{
    var arr = new string[22];
    for (int i = 0; i < 22; i++)
        arr[i] = $"BIST_PT_ORD{i}";
    return arr;
}

====

var ordersObj = autoRunControl["orders"] as JObject;
var fromJson  = ordersObj?["targets"]?.ToObject<string[]>();
if (fromJson != null && fromJson.Length > 0)
{
    _orderTargets = fromJson;
    System.Diagnostics.Debug.WriteLine($"[AutoRun] Loaded {_orderTargets.Length} order targets from JSON.");
}
else
{
    System.Diagnostics.Debug.WriteLine("[AutoRun] No order targets found in JSON, using fallback.");
}

====
int desired = Total;
if (desired < 1) desired = 1;
if (desired > 22) desired = 22;

int count = Math.Min(desired, _orderTargets.Length);

for (int i = 0; i < count; i++)
{
    string target = _orderTargets[i];
    if (string.IsNullOrEmpty(target)) continue;
    byte code = (byte)(Orders[i].SelectedIndex & 0x3F);
    RegMap.Write8(target, code);
}