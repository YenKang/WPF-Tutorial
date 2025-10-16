public void LoadFrom(object jsonBlock)
{
    var j = jsonBlock as Newtonsoft.Json.Linq.JObject;
    if (j == null) return;

    var f = j["fields"] as Newtonsoft.Json.Linq.JObject;

    D2Options.Clear();
    for (int i = (int)f["D2"]["min"]; i <= (int)f["D2"]["max"]; i++)
        D2Options.Add(i);

    D1Options.Clear();
    for (int i = (int)f["D1"]["min"]; i <= (int)f["D1"]["max"]; i++)
        D1Options.Add(i);

    D0Options.Clear();
    for (int i = (int)f["D0"]["min"]; i <= (int)f["D0"]["max"]; i++)
        D0Options.Add(i);

    // ✅ 設定 default 值
    D2 = (int)f["D2"]["default"];
    D1 = (int)f["D1"]["default"];
    D0 = (int)f["D0"]["default"];
}