public void DebugJsonAndMapSmoke()
{
    if (_profile == null)
    {
        System.Windows.MessageBox.Show("❌ profile = null");
        return;
    }

    // 1️⃣ JSON 結構檢查
    string msg =
        "IcName = " + _profile.IcName + "\n" +
        "Registers = " + (_profile.Registers != null ? _profile.Registers.Count.ToString() : "null") + "\n" +
        "Patterns = " + (_profile.Patterns != null ? _profile.Patterns.Count.ToString() : "null");
    System.Diagnostics.Debug.WriteLine("✅ JSON OK:\n" + msg);

    // 2️⃣ 檢查 RegMap 是否已初始化
    try
    {
        var pr = RegMap.Get("BIST_PT_LEVEL"); // 僅取位址，不實際存取硬體
        System.Diagnostics.Debug.WriteLine(
            "✅ RegMap Get(BIST_PT_LEVEL) => page=" + pr.Item1 + " reg=" + pr.Item2);

        System.Windows.MessageBox.Show("✅ JSON + RegMap 正常！");
    }
    catch (System.Exception ex)
    {
        System.Windows.MessageBox.Show("⚠️ RegMap 尚未初始化或找不到 key：\n" + ex.Message);
    }
}
