var j = grayLevelControl as JObject;
var f = j?["fields"] as JObject;
System.Diagnostics.Debug.WriteLine($"[GrayVM] fields is null? { (f==null) }");

System.Diagnostics.Debug.WriteLine($"[GrayVM] D2Options={D2Options.Count}, 
D1={D1Options.Count}, D0={D0Options.Count}");

public void LoadFrom(object grayLevelControl)
{
    var j = grayLevelControl as JObject;
    if (j == null) return;

    var f = j["fields"] as JObject;
    if (f == null)
    {
        System.Diagnostics.Debug.WriteLine("[GrayVM] fields missing in JSON.");
        return; // ⛔ 不清空，保留原本選項，避免空白造成 '' → int 的錯誤
    }

    ApplyFieldRange(D2Options, f["D2"] as JObject, 0, 3,  v => D2 = v);
    ApplyFieldRange(D1Options, f["D1"] as JObject, 0, 15, v => D1 = v);
    ApplyFieldRange(D0Options, f["D0"] as JObject, 0, 15, v => D0 = v);

    System.Diagnostics.Debug.WriteLine($"[GrayVM] Done. D2={D2}, D1={D1}, D0={D0}");
}

private static void ApplyFieldRange(
    ObservableCollection<int> target,
    JObject field,
    int fallbackMin, int fallbackMax,
    System.Action<int> setDefault)
{
    // 如果該欄位缺失，用 fallback；這樣不會把清單清到空
    int min = (int?)field?["min"] ?? fallbackMin;
    int max = (int?)field?["max"] ?? fallbackMax;
    int def = (int?)field?["default"] ?? min;

    target.Clear();
    for (int i = min; i <= max; i++) target.Add(i);

    if (def < min) def = min;
    if (def > max) def = max;
    setDefault(def); // ← 最後設定 SelectedItem 對應的值
}