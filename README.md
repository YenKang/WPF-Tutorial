using BistMode.MVVM; // 有 GetValue/SetValue 的基底
using Newtonsoft.Json.Linq;
using System.Collections.ObjectModel;

public class GrayColorVM : ViewModelBase
{
    private JObject _cfg;
    public ObservableCollection<int> BoolOptions { get; private set; } =
        new ObservableCollection<int>(new int[] { 0, 1 });

    public int R
    {
        get { return GetValue<int>(); }
        set { SetValue(value); } // ✅ 自動 Raise PropertyChanged
    }

    public int G
    {
        get { return GetValue<int>(); }
        set { SetValue(value); }
    }

    public int B
    {
        get { return GetValue<int>(); }
        set { SetValue(value); }
    }

    public ICommand ApplyCommand { get; private set; }

    public GrayColorVM()
    {
        ApplyCommand = CommandFactory.CreateCommand(ExecuteWrite);
    }

    public void LoadFrom(object jsonCfg)
    {
        _cfg = jsonCfg as JObject;
        var f = _cfg?["fields"] as JObject;
        if (f == null) return;

        R = ClampDefault(f["R"] as JObject, 0, 1, 1);  // default 1
        G = ClampDefault(f["G"] as JObject, 0, 1, 0);
        B = ClampDefault(f["B"] as JObject, 0, 1, 0);
    }

    private static int ClampDefault(JObject f, int min, int max, int def)
    {
        if (f == null) return def;
        int vmin = (int?)f["min"] ?? min;
        int vmax = (int?)f["max"] ?? max;
        int vdef = (int?)f["default"] ?? def;
        if (vdef < vmin) vdef = vmin;
        if (vdef > vmax) vdef = vmax;
        return vdef;
    }

    private void ExecuteWrite()
    {
        // 寫入 004F bit[2:0]
        byte val = (byte)((B & 0x1) << 2 | (G & 0x1) << 1 | (R & 0x1));
        RegMap.Write8("BIST_GrayColor_VH_Reverse", val);
    }
}