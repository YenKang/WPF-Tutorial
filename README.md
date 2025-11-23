var regObj = autoRunNode["patternOrder"]?["register"] as JObject;
if (regObj != null)
{
    foreach (var prop in regObj.Properties())
    {
        string regName = prop.Name;
        int defaultVal = prop.Value.Value<int>();  // ← 解析正確

        m.OrderRegs.Add(regName);
        m.OrderValues.Add(defaultVal);

        Debug.WriteLine($"[AutoRun] OrderReg={regName}, Default={defaultVal}");
    }
}
