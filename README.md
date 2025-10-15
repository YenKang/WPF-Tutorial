public sealed class RegDef
{
    public string addr { get; set; }
    public int width { get; set; }
    public string desc { get; set; }
}

public sealed class FieldDef
{
    public int min { get; set; }
    public int max { get; set; }

    [Newtonsoft.Json.JsonProperty("default")]
    public int defaultValue { get; set; }
}

public sealed class RowDef
{
    public string id { get; set; }
    public string title { get; set; }
    public string fieldType { get; set; }
    public Dictionary<string, FieldDef> fields { get; set; }
    public System.Collections.Generic.List<object> writes { get; set; }
}

public sealed class PatternDef
{
    public string name { get; set; }
    public int index { get; set; }
    public System.Collections.Generic.List<RowDef> rows { get; set; }
}

public sealed class ChipProfile
{
    public string chip { get; set; }
    public System.Collections.Generic.Dictionary<string, RegDef> registers { get; set; }
    public System.Collections.Generic.List<PatternDef> patterns { get; set; }
}



=======

public static class ProfileLoaderNew
{
    public static ChipProfile LoadChipProfile(string path)
    {
        if (!File.Exists(path))
            throw new FileNotFoundException("找不到指定 JSON Profile。", path);

        string json = File.ReadAllText(path);

        ChipProfile profile = JsonConvert.DeserializeObject<ChipProfile>(json);

        if (profile == null)
            throw new InvalidOperationException("ChipProfile 反序列化失敗。");

        if (profile.registers == null)
            throw new InvalidOperationException("registers 欄位缺失。");

        if (profile.patterns == null)
            throw new InvalidOperationException("patterns 欄位缺失。");

        return profile;
    }

    public static ushort ToAddr(string hex)
    {
        if (string.IsNullOrWhiteSpace(hex))
            throw new ArgumentException("位址字串為空。");
        if (hex.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            hex = hex.Substring(2);
        return Convert.ToUInt16(hex, 16);
    }

    public static int ParseInt(string s)
    {
        if (s == null)
            throw new ArgumentNullException("s");
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            return Convert.ToInt32(s.Substring(2), 16);
        return int.Parse(s);
    }
}

===========
