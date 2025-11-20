for (int cp = 1; cp <= 7; cp++)
{
    foreach (var ch in new[] { "R", "G", "B" })
    {
        for (int lv = 1; lv <= 7; lv++)
        {
            ColorPaletteRegList.Add(new ColorPaletteReg
            {
                RegName = $"ICON_CP{cp}_{ch}{lv:00}",
                Data = 0
            });
        }
    }
}

＝＝＝
public class ColorPaletteReg
{
    public string RegName;   // e.g. "ICON_CP1_R03"
    public uint Data;        // 最終 register 數值
}

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
public void ApplyColorPalette(Action<string, uint> writeReg)
{
    foreach (var cp in ColorPaletteRegList)
    {
        writeReg(cp.RegName, cp.Data);
    }
}