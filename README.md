using Newtonsoft.Json.Linq;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Globalization;

public class GrayLevelVM : INotifyPropertyChanged
{
    public ObservableCollection<int> D2Options { get; } = new ObservableCollection<int>();
    public ObservableCollection<int> D1Options { get; } = new ObservableCollection<int>();
    public ObservableCollection<int> D0Options { get; } = new ObservableCollection<int>();

    // 建議用 nullable，避免載入過程出現 ""→int 的紅框
    private int? _d2; public int? D2 { get => _d2; set { if (_d2 == value) return; _d2 = value; OnPropertyChanged(); } }
    private int? _d1; public int? D1 { get => _d1; set { if (_d1 == value) return; _d1 = value; OnPropertyChanged(); } }
    private int? _d0; public int? D0 { get => _d0; set { if (_d0 == value) return; _d0 = value; OnPropertyChanged(); } }

    public void LoadFrom(object jsonCfg)
    {
        var j = jsonCfg as JObject;
        var f = j?["fields"] as JObject;
        if (f == null) return;

        // 1) 先清掉選取（防止 SelectedItem 指向不存在項目）
        D2 = null; D1 = null; D0 = null;

        // 2) 每個 Digit 依 JSON 的 min/max 產生選項，最後套 default
        ApplyFieldRange(D2Options, f["D2"] as JObject, 0, 3,  v => D2 = v);
        ApplyFieldRange(D1Options, f["D1"] as JObject, 0, 15, v => D1 = v);
        ApplyFieldRange(D0Options, f["D0"] as JObject, 0, 15, v => D0 = v);

        // Debug 檢查
        System.Diagnostics.Debug.WriteLine(
            $"[GL] D2 {D2Options.Count} | D1 {D1Options.Count} | D0 {D0Options.Count}  // Selected: {D2},{D1},{D0}");
    }

    private static void ApplyFieldRange(
        ObservableCollection<int> target,
        JObject field,
        int fallbackMin, int fallbackMax,
        System.Action<int> setDefault)
    {
        int min = GetInt(field, "min", fallbackMin);
        int max = GetInt(field, "max", fallbackMax);
        int def = GetInt(field, "default", min);

        if (min > max) { var t = min; min = max; max = t; }
        if (def < min) def = min;
        if (def > max) def = max;

        // 不更換集合實例，直接重建內容
        target.Clear();
        for (int i = min; i <= max; i++) target.Add(i);

        // Items 準備好之後再設定 Selected
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
            if (s.StartsWith("0x", System.StringComparison.OrdinalIgnoreCase))
            {
                if (int.TryParse(s.Substring(2), NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var hv)) return hv;
            }
            if (int.TryParse(s, NumberStyles.Integer, CultureInfo.InvariantCulture, out var dv)) return dv;
        }
        return fallback;
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void OnPropertyChanged([CallerMemberName] string name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}