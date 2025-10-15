using System.Collections.Generic;

public class ProfileRoot
{
    public List<PatternNode> patterns { get; set; } = new List<PatternNode>();
}

public class PatternNode
{
    public int index { get; set; }
    public string name { get; set; }
    public string icon { get; set; }
    public List<ConfigUIRaw> configUIRaws { get; set; } = new List<ConfigUIRaw>();
}

public class ConfigUIRaw
{
    public string id { get; set; }
    public string title { get; set; }
    public string kind { get; set; }
    public Dictionary<string, FieldDef> fields { get; set; } = new Dictionary<string, FieldDef>();
    public List<WriteOp> writes { get; set; } = new List<WriteOp>();
    public string explain { get; set; }
}

public class FieldDef
{
    public int min { get; set; }
    public int max { get; set; }
    public int @default { get; set; }  // "default"
}

public class WriteOp
{
    public string mode { get; set; }
    public string target { get; set; }
    public string mask { get; set; }
    public int? shift { get; set; }
    public string src { get; set; }
}