using System;
using System.Collections.ObjectModel;
using System.Globalization;
using System.Windows.Input;
using Newtonsoft.Json.Linq;

public class GrayColorVM
{
    private JObject _cfg;  // 記住 grayColorControl 的 JSON 區塊（fields / writes）

    // 下拉固定 0/1（不需要重建）
    public ObservableCollection<int> BoolOptions { get; } = new ObservableCollection<int>(new[] { 0, 1 });

    // ✅ 依照你 GrayLevelVM 的風格：不觸發 OnPropertyChanged，交給你底層機制
    public int R { get; set; }  // bist_gray_r_en
    public int G { get; set; }  // bist_gray_g_en
    public int B { get; set; }  // bist_gray_b_en

    public ICommand ApplyCommand { get; }

    public GrayColorVM()
    {
        ApplyCommand = new RelayCommand(_ => ExecuteWrite());
        // 若需要預設，可在這裡給，不過通常由 JSON default 決定
        // R = 1; G = 0; B = 0;
    }

    public void LoadFrom(object jsonCfg)
    {
        _cfg = jsonCfg as JObject;

        var f = _cfg?["fields"] as JObject;
        if (f == null) return;

        // 依 JSON default 設定（不使用 OnPropertyChanged）
        R = ClampDefault(f["R"] as JObject, 0, 1, 0);
        G = ClampDefault(f["G"] as JObject, 0, 1, 0);
        B = ClampDefault(f["B"] as JObject, 0, 1, 0);

        System.Diagnostics.Debug.WriteLine($"[GrayColor] defaults => R={R}, G={G}, B={B}");
    }

    private static int ClampDefault(JObject field, int fbMin, int fbMax, int fbDef)
    {
        int min = GetInt(field, "min", fbMin);
        int max = GetInt(field, "max", fbMax);
        int def = GetInt(field, "default", fbDef);
        if (min > max) { var t = min; min = max; max = t; }
        if (def < min) def = min;
        if (def > max) def = max;
        // BoolOptions 固定 0/1，不用更新 ItemsSource
        return def;
    }

    private static int GetInt(JObject obj, string key, int fallback)
    {
        if (obj == null) return fallback;
        var tok = obj[key];
        if (tok == null) return fallback;

        if (tok.Type == JTokenType.Integer) return tok.Value<int>();
        if (tok.Type == JTokenType.String)
        {
            var s = tok.Value<string>().Trim();
            int v;
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            {
                if (int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v))
                    return v;
            }
            if (int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v))
                return v;
        }
        return fallback;
    }

    private void ExecuteWrite()
    {
        // 將 R/G/B 各取 1bit
        int r = R & 0x1;
        int g = G & 0x1;
        int b = B & 0x1;

        var writes = _cfg?["writes"] as JArray;
        if (writes == null || writes.Count == 0)
        {
            // Fallback：一次 RMW 寫 0x004F（或你在 RegMap 中命名的新暫存器）
            // 若你已改名為 "BIST_GrayColor_VH_Reverse"，用那個 target 名稱
            byte cur = RegMap.Read8("BIST_GrayColor_VH_Reverse");
            byte next = (byte)((cur & ~0x07) | (r << 0) | (g << 1) | (b << 2));
            RegMap.Write8("BIST_GrayColor_VH_Reverse", next);
            return;
        }

        // 依 JSON 的 writes 動態執行
        foreach (JObject w in writes)
        {
            string mode = (string)w["mode"] ?? "rmw";
            string target = (string)w["target"];
            if (string.IsNullOrEmpty(target)) continue;

            int srcVal = ResolveSrc((string)w["src"], r, g, b);

            if (mode == "write8")
            {
                RegMap.Write8(target, (byte)(srcVal & 0xFF));
            }
            else if (mode == "rmw")
            {
                int mask  = ParseInt((string)w["mask"]);
                int shift = (int?)w["shift"] ?? 0;

                byte cur  = RegMap.Read8(target);
                byte next = (byte)((cur & ~(mask & 0xFF)) | (((srcVal << shift) & mask) & 0xFF));
                RegMap.Write8(target, next);
            }
        }
    }

    // C# 7.3 相容版本（不用 switch expression）
    private static int ResolveSrc(string src, int r, int g, int b)
    {
        if (string.IsNullOrEmpty(src)) return 0;

        switch (src)
        {
            case "R": return r;
            case "G": return g;
            case "B": return b;
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
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
        return int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v);
    }
    private static int ParseInt(string s) { int v; return TryParseInt(s, out v) ? v : 0; }
}

// 你專案裡已有 RelayCommand/CommandFactory 就用你的；否則用這個最小版
public class RelayCommand : ICommand
{
    private readonly Action<object> _run; private readonly Func<object, bool> _can;
    public RelayCommand(Action<object> run, Func<object, bool> can = null) { _run = run; _can = can; }
    public bool CanExecute(object p) => _can == null || _can(p);
    public void Execute(object p) => _run(p);
    public event EventHandler CanExecuteChanged { add { } remove { } }
}