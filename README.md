using System.Globalization;
using Newtonsoft.Json.Linq;
using System.Windows.Input;
using System.Collections.ObjectModel;

public class GrayColorVM : ViewModelBase
{
    private JObject _cfg;

    public ObservableCollection<int> BoolOptions { get; } =
        new ObservableCollection<int>(new[] { 0, 1 });

    // 用 ViewModelBase 的 GetValue/SetValue（會自動通知 UI）
    public int R { get { return GetValue<int>(); } set { SetValue(value); } }
    public int G { get { return GetValue<int>(); } set { SetValue(value); } }
    public int B { get { return GetValue<int>(); } set { SetValue(value); } }

    public ICommand ApplyCommand { get; }

    public GrayColorVM()
    {
        ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
    }

    public void LoadFrom(object jsonCfg)
    {
        _cfg = jsonCfg as JObject;
        var f = _cfg?["fields"] as JObject;
        if (f == null) return;

        R = ClampDefault(f["R"] as JObject, 0, 1, 1); // 依 JSON default
        G = ClampDefault(f["G"] as JObject, 0, 1, 0);
        B = ClampDefault(f["B"] as JObject, 0, 1, 0);
    }

    private static int ClampDefault(JObject field, int fbMin, int fbMax, int fbDef)
    {
        if (field == null) return fbDef;
        int min = (int?)field["min"] ?? fbMin;
        int max = (int?)field["max"] ?? fbMax;
        int def = (int?)field["default"] ?? fbDef;
        if (min > max) { var t = min; min = max; max = t; }
        if (def < min) def = min;
        if (def > max) def = max;
        return def;
    }

    // ★★★ 這裡加入 mask/shift 的動態寫入 ★★★
    private void ExecuteWrite()
    {
        int r = R & 0x1, g = G & 0x1, b = B & 0x1;
        var writes = _cfg?["writes"] as JArray;

        if (writes == null || writes.Count == 0)
        {
            // Fallback：一次 RMW 寫入 0x004F 的 bit[2:0]
            byte cur  = RegMap.Read8("BIST_GrayColor_VH_Reverse");
            byte next = (byte)((cur & ~0x07) | (r << 0) | (g << 1) | (b << 2));
            RegMap.Write8("BIST_GrayColor_VH_Reverse", next);
            return;
        }

        foreach (JObject w in writes)
        {
            string mode   = (string)w["mode"]   ?? "rmw";
            string target = (string)w["target"];     if (string.IsNullOrEmpty(target)) continue;
            string srcStr = (string)w["src"];        // "R"/"G"/"B"/"RGBPacked"/"1"/"0x03"…
            int    srcVal = ResolveSrc(srcStr, r, g, b);

            if (mode == "write8")
            {
                RegMap.Write8(target, (byte)(srcVal & 0xFF));
                continue;
            }

            // rmw
            int mask  = ParseInt((string)w["mask"]);   // 支援 "0x.." 或十進位
            int shift = (int?)w["shift"] ?? 0;

            byte cur  = RegMap.Read8(target);
            byte next = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
            RegMap.Write8(target, next);
        }
    }

    // C# 7.3 相容版本（傳統 switch）
    private static int ResolveSrc(string src, int r, int g, int b)
    {
        if (string.IsNullOrEmpty(src)) return 0;

        switch (src)
        {
            case "R": return r;
            case "G": return g;
            case "B": return b;
            case "RGBPacked": return (r & 0x1) | ((g & 0x1) << 1) | ((b & 0x1) << 2);
            default:
                int v;
                if (TryParseInt(src, out v)) return v; // 支援常數
                return 0;
        }
    }

    private static bool TryParseInt(string s, out int v)
    {
        v = 0;
        if (string.IsNullOrWhiteSpace(s)) return false;
        s = s.Trim();
        if (s.StartsWith("0x", System.StringComparison.OrdinalIgnoreCase))
            return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
        return int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v);
    }

    private static int ParseInt(string s) { int v; return TryParseInt(s, out v) ? v : 0; }
}