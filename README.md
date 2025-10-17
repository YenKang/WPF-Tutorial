{
  "index": 11,
  "name": "Cursor",
  "regControlType": ["grayLevelControl", "grayColorControl"],

  "grayColorControl": {
    "title": "Gray Level Color",
    "fields": {
      "R": { "min": 0, "max": 1, "default": 1 },
      "G": { "min": 0, "max": 1, "default": 0 },
      "B": { "min": 0, "max": 1, "default": 0 }
    },
    "writes": [
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x01", "shift": 0, "src": "R" },  // bit0
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x02", "shift": 1, "src": "G" },  // bit1
      { "mode": "rmw", "target": "BIST_GRAY_COLOR", "mask": "0x04", "shift": 2, "src": "B" }   // bit2
    ]
  }
}



using System;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Globalization;
using System.Windows.Input;
using Newtonsoft.Json.Linq;

public class GrayColorVM : INotifyPropertyChanged
{
    private JObject _cfg;   // 記住 grayColorControl JSON 區塊（fields & writes）

    // 0/1 選項（固定）
    public ObservableCollection<int> BoolOptions { get; } = new ObservableCollection<int>(new[] { 0, 1 });

    // 三個選值（用 nullable，避免載入過程紅框）
    private int? _r; public int? R { get => _r; set { if (_r == value) return; _r = value; OnPropertyChanged(); } }
    private int? _g; public int? G { get => _g; set { if (_g == value) return; _g = value; OnPropertyChanged(); } }
    private int? _b; public int? B { get => _b; set { if (_b == value) return; _b = value; OnPropertyChanged(); } }

    public ICommand ApplyCommand { get; }

    public GrayColorVM()
    {
        ApplyCommand = new RelayCommand(_ => ExecuteWrite());
    }

    public void LoadFrom(object jsonCfg)
    {
        _cfg = jsonCfg as JObject;
        var f = _cfg?["fields"] as JObject;
        if (f == null) return;

        // 過渡期先清空，避免 SelectedItem 寫回空字串
        R = null; G = null; B = null;

        ApplyField("R", f["R"] as JObject, v => R = v);
        ApplyField("G", f["G"] as JObject, v => G = v);
        ApplyField("B", f["B"] as JObject, v => B = v);

        System.Diagnostics.Debug.WriteLine($"[GrayColor] R={R}, G={G}, B={B}");
    }

    private void ApplyField(string name, JObject field, Action<int> setDefault)
    {
        int min = GetInt(field, "min", 0);
        int max = GetInt(field, "max", 1);
        int def = GetInt(field, "default", 0);
        if (min > max) { var t = min; min = max; max = t; }
        if (def < min) def = min;
        if (def > max) def = max;

        // BoolOptions 是固定 0/1，不需重建 ItemsSource，只要把選中值設為 default
        setDefault(def);
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
            if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase) &&
                int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var hv)) return hv;
            if (int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out var dv)) return dv;
        }
        return fallback;
    }

    private void ExecuteWrite()
    {
        int r = (R ?? 0) & 0x1;
        int g = (G ?? 0) & 0x1;
        int b = (B ?? 0) & 0x1;

        var writes = _cfg?["writes"] as JArray;
        if (writes == null || writes.Count == 0)
        {
            // Fallback：一次 RMW 寫 0x004F 的 bit2..0
            byte cur = RegMap.Read8("BIST_GRAY_COLOR");
            byte next = (byte)((cur & ~0x07) | (r << 0) | (g << 1) | (b << 2));
            RegMap.Write8("BIST_GRAY_COLOR", next);
            return;
        }

        // 依 JSON 執行（可自由改 mask/shift/target/src）
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

    private static int ResolveSrc(string src, int r, int g, int b)
    {
        if (string.IsNullOrEmpty(src)) return 0;
        return src switch
        {
            "R" => r, "G" => g, "B" => b,
            _ => (TryParseInt(src, out var v) ? v : 0) // 也允許常數
        };
    }

    private static bool TryParseInt(string s, out int v)
    {
        v = 0; if (string.IsNullOrWhiteSpace(s)) return false;
        s = s.Trim();
        if (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            return int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out v);
        return int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out v);
    }
    private static int ParseInt(string s) => TryParseInt(s, out var v) ? v : 0;

    public event PropertyChangedEventHandler PropertyChanged;
    private void OnPropertyChanged([CallerMemberName] string n = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(n));
}

// 若你專案已有 RelayCommand/CommandFactory，請改用你的；否則用這個最小版
public class RelayCommand : ICommand
{
    private readonly Action<object> _run; private readonly Func<object, bool> _can;
    public RelayCommand(Action<object> run, Func<object, bool> can = null) { _run = run; _can = can; }
    public bool CanExecute(object p) => _can == null || _can(p);
    public void Execute(object p) => _run(p);
    public event EventHandler CanExecuteChanged { add { } remove { } }
}