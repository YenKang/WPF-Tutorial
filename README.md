using Newtonsoft.Json.Linq;
using System;
using System.Globalization;
using System.Windows.Input;

public class GrayReverseVM : ViewModelBase
{
    private JObject _cfg;       // 記住 JSON block
    private string _axis = "H"; // "H" or "V"

    public string Title
    {
        get { return _axis == "V" ? "Vertical Gray Reverse Enable" : "Horizontal Gray Reverse Enable"; }
    }

    // Checkbox 綁定的值（true=勾）
    public bool Enable
    {
        get { return GetValue<bool>(); }
        set { SetValue(value); }
    }

    public ICommand ApplyCommand { get; }

    public GrayReverseVM()
    {
        ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
        // 預設（若未載入 JSON）
        _axis = "H";
        Enable = false;
    }

    public void LoadFrom(object jsonCfg)
    {
        _cfg = jsonCfg as JObject;
        if (_cfg == null) return;

        var axisToken = _cfg["axis"];
        var axis = axisToken != null ? axisToken.ToString().Trim().ToUpperInvariant() : "H";
        _axis = (axis == "V") ? "V" : "H";
        OnPropertyChanged(nameof(Title)); // 更新標題

        // default: 0/1
        int def = 0;
        var defTok = _cfg["default"];
        if (defTok != null) int.TryParse(defTok.ToString(), out def);
        Enable = (def != 0);
    }

    private void ExecuteWrite()
    {
        // 優先走 JSON 的 writes（宣告式）
        var writes = _cfg?["writes"] as JArray;
        if (writes != null && writes.Count > 0)
        {
            foreach (JObject w in writes)
            {
                string mode   = (string)w["mode"]   ?? "rmw";
                string target = (string)w["target"];
                if (string.IsNullOrEmpty(target)) continue;

                int srcVal = ResolveSrc((string)w["src"]); // 0/1
                if (mode == "write8")
                {
                    RegMap.Write8(target, (byte)(srcVal & 0xFF));
                }
                else
                {
                    int mask  = ParseInt((string)w["mask"]);
                    int shift = (int?)w["shift"] ?? 0;
                    byte cur  = RegMap.Read8(target);
                    byte nxt  = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
                    RegMap.Write8(target, nxt);
                }
            }
            return;
        }

        // 沒有 writes 時的後援（直接針對 004Fh）
        string reg = "BIST_GrayColor_VH_Reverse";
        int bit = (_axis == "V") ? 5 : 4;
        byte curVal = RegMap.Read8(reg);
        byte newVal = Enable ? (byte)(curVal |  (1 << bit))
                             : (byte)(curVal & ~(1 << bit));
        RegMap.Write8(reg, newVal);
    }

    // 支援 "enable" 或常數
    private int ResolveSrc(string src)
    {
        if (string.IsNullOrEmpty(src)) return Enable ? 1 : 0;
        if (src == "enable") return Enable ? 1 : 0;

        int v;
        return TryParseInt(src, out v) ? v : (Enable ? 1 : 0);
    }

    private static bool TryParseInt(string s, out int v)
    {
        v = 0;
        if (string.IsNullOrWhiteSpace(s)) return false;
        s = s.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
        return int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v);
    }
    private static int ParseInt(string s) { int v; return TryParseInt(s, out v) ? v : 0; }
}