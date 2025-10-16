private static void ApplyFieldRange(
    ObservableCollection<int> target,
    Newtonsoft.Json.Linq.JObject field,
    int fallbackMin, int fallbackMax,
    System.Action<int> setDefault)
{
    // 1) 讀 JSON（若 field 缺，就用 fallback）
    int min = fallbackMin, max = fallbackMax, def = fallbackMin;
    if (field != null)
    {
        var tMin = field["min"];  if (tMin != null) min = tMin.Value<int>();
        var tMax = field["max"];  if (tMax != null) max = tMax.Value<int>();
        var tDef = field["default"]; if (tDef != null) def = tDef.Value<int>();
    }

    // 2) 夾值
    if (min > max) { var tmp = min; min = max; max = tmp; }
    if (def < min) def = min;
    if (def > max) def = max;

    // 3) 先把選取清掉（避免舊值殘留導致 SelectedIndex 偏移）
    setDefault(min);  // 先設一個合法值，或你也可以先設 null（若 D2 是 int?）

    // 4) 填入 Items（不 new；只 Clear+Add）
    target.Clear();
    for (int i = min; i <= max; i++) target.Add(i);

    // 5) 最後才設 default，這時 Items 已就緒 → SelectedItem 會正確命中 def
    setDefault(def);

    System.Diagnostics.Debug.WriteLine(
        $"[ApplyFieldRange] min={min} max={max} def={def} -> items={target.Count}");
}

public void LoadFrom(object json)
{
    var j = json as Newtonsoft.Json.Linq.JObject;
    if (j == null) return;
    var f = j["fields"] as Newtonsoft.Json.Linq.JObject;
    if (f == null) return;

    ApplyFieldRange(D2Options, f["D2"] as Newtonsoft.Json.Linq.JObject, 0, 3,  v => D2 = v);
    ApplyFieldRange(D1Options, f["D1"] as Newtonsoft.Json.Linq.JObject, 0, 15, v => D1 = v);
    ApplyFieldRange(D0Options, f["D0"] as Newtonsoft.Json.Linq.JObject, 0, 15, v => D0 = v);

    System.Diagnostics.Debug.WriteLine($"[LoadFrom] Selected D2={D2}, D1={D1}, D0={D0}");
}