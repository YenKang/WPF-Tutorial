//------------------------------
// patternOrder (動態 N 筆)
//------------------------------
m.OrderRegs.Clear();
m.OrderValues.Clear();

// register 是「陣列」不是 JObject
var regArray = autoRunNode["patternOrder"]?["register"] as JArray;

if (regArray != null)
{
    foreach (var jo in regArray.OfType<JObject>())
    {
        // 每一格只有一個屬性：{ "BIST_PT_ORD0": 0 }
        var prop = jo.Properties().FirstOrDefault();
        if (prop == null)
            continue;

        string regName  = prop.Name;                 // "BIST_PT_ORD0"
        int defaultVal  = prop.Value.Value<int>();   // 0, 1, 2…

        m.OrderRegs.Add(regName);
        m.OrderValues.Add(defaultVal);

        Debug.WriteLine($"[AutoRun] OrderReg={regName}, Default={defaultVal}");
    }
}
else
{
    Debug.WriteLine("[AutoRun] patternOrder.register is NULL");
}
