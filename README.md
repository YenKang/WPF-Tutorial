// 新結構原本就有的欄位：
// public string chip { get; set; }
// public Dictionary<string, RegDef> registers { get; set; }
// public List<PatternDef> patterns { get; set; }

// === 下面是「舊名相容」屬性（唯讀，轉接到新欄位） ===

// 舊碼常用：IcName → 對應新 chip
public string IcName
{
    get { return this.chip; }
}

// 舊碼常用：Patterns（PascalCase） → 對應新 patterns
public System.Collections.Generic.List<PatternDef> Patterns
{
    get { return this.patterns; }
}

// 舊碼常用：Registers（Dictionary<string,string>） → 從新 registers 取 addr 轉一份字串表
public System.Collections.Generic.Dictionary<string, string> Registers
{
    get
    {
        var map = new System.Collections.Generic.Dictionary<string, string>();
        if (this.registers != null)
        {
            foreach (var kv in this.registers)
            {
                // kv.Key: 名稱（如 "BIST_PT_LEVEL"）
                // kv.Value.addr: 位址（如 "0x0030"）
                map[kv.Key] = (kv.Value != null) ? kv.Value.addr : null;
            }
        }
        return map;
    }
}