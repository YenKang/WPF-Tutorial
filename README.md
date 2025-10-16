using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Windows.Input;
using Newtonsoft.Json.Linq;

public class GrayLevelVM
{
    public ObservableCollection<int> D2Options { get; private set; } = new ObservableCollection<int>(new[] { 0,1,2,3 });
    public ObservableCollection<int> D1Options { get; private set; } = new ObservableCollection<int>(Generate(0,15));
    public ObservableCollection<int> D0Options { get; private set; } = new ObservableCollection<int>(Generate(0,15));

    public int D2 { get; set; } = 3;
    public int D1 { get; set; } = 15;
    public int D0 { get; set; } = 15;

    public ICommand ApplyCommand { get; private set; }

    public GrayLevelVM()
    {
        ApplyCommand = new RelayCommand(_ => ExecuteWrite());
    }

    // 從 JSON 的 grayLevelControl 區塊載入
    public void LoadFrom(object grayLevelControl)
    {
        var j = grayLevelControl as JObject;
        if (j == null) return;

        var f = j["fields"] as JObject;
        if (f != null)
        {
            ApplyFieldRange(D2Options, f["D2"] as JObject, 0, 3, v => D2 = v);
            ApplyFieldRange(D1Options, f["D1"] as JObject, 0, 15, v => D1 = v);
            ApplyFieldRange(D0Options, f["D0"] as JObject, 0, 15, v => D0 = v);
        }
    }

    private static void ApplyFieldRange(ObservableCollection<int> target, JObject field, int defMin, int defMax, System.Action<int> setDefault)
    {
        int min = defMin, max = defMax, d = defMin;
        if (field != null)
        {
            min = (int?)field["min"] ?? defMin;
            max = (int?)field["max"] ?? defMax;
            d   = (int?)field["default"] ?? min;
        }

        target.Clear();
        for (int i = min; i <= max; i++) target.Add(i);
        setDefault(d);
    }

    private void ExecuteWrite()
    {
        // 組 10-bit 值：D2[1:0]|D1[3:0]|D0[3:0]
        int gray10 = ((D2 & 0x3) << 8) | ((D1 & 0xF) << 4) | (D0 & 0xF);
        byte low = (byte)(gray10 & 0xFF);

        // 寫 BIST_PT_LEVEL 的低位元組
        RegMap.Write8("BIST_PT_LEVEL", low);

        // RMW BIST_CK_GY_FK_PT 的低 2 bits（D2）
        byte cur = RegMap.Read8("BIST_CK_GY_FK_PT");
        byte newVal = (byte)((cur & ~0x03) | (D2 & 0x03));
        RegMap.Write8("BIST_CK_GY_FK_PT", newVal);
    }

    private static IEnumerable<int> Generate(int a, int b)
    {
        for (int i = a; i <= b; i++) yield return i;
    }
}

// 簡易 RelayCommand（若你專案已有可省略）
public class RelayCommand : ICommand
{
    private readonly System.Action<object> _exec; private readonly System.Func<object,bool> _can;
    public RelayCommand(System.Action<object> exec, System.Func<object,bool> can = null) { _exec = exec; _can = can; }
    public bool CanExecute(object p) => _can == null || _can(p);
    public void Execute(object p) => _exec(p);
    public event System.EventHandler CanExecuteChanged { add { } remove { } }
}