using System.Collections.Generic;
using System.ComponentModel;
using System.Runtime.CompilerServices;

public class ConfigUIRaw
{
    public string id { get; set; }
    public string title { get; set; }
    public string kind { get; set; }
    public Dictionary<string, FieldDef> fields { get; set; }
    public List<WriteOp> writes { get; set; }
    public string explain { get; set; }
}

public class WriteOp
{
    public string mode { get; set; }   // "write8" / "rmw"
    public string target { get; set; } // 如 "BIST_PT_LEVEL"
    public string mask { get; set; }   // "0x03" or "3"（rmw 用）
    public int?   shift { get; set; }  // 0（rmw 用）
    public string src { get; set; }    // "LowByte"/"HighByte"/"D2"/"D1"/"D0"
}

public class FieldDef : INotifyPropertyChanged
{
    private int _min, _max, _def;
    public int min { get { return _min; } set { if (_min==value) return; _min=value; OnPropertyChanged(); OnPropertyChanged("Options"); } }
    public int max { get { return _max; } set { if (_max==value) return; _max=value; OnPropertyChanged(); OnPropertyChanged("Options"); } }
    public int @default { get { return _def; } set { if (_def==value) return; _def=value; OnPropertyChanged(); OnPropertyChanged("value"); } }

    // 給 XAML 綁 TwoWay 用
    public int value { get { return @default; } set { if (_def==value) return; _def=value; OnPropertyChanged(); OnPropertyChanged("@default"); } }

    public IEnumerable<int> Options { get { for (var i=min; i<=max; i++) yield return i; } }

    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string n=null)
    { var h=PropertyChanged; if (h!=null) h(this,new PropertyChangedEventArgs(n)); }
}