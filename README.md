using System;
using System.Collections.ObjectModel;
using System.Collections.Generic;
using System.ComponentModel;
using System.IO;
using System.Linq;
using System.Text.Json;
using System.Windows.Input;

public class BistModeViewModel : INotifyPropertyChanged
{
    private ProfileRoot _root;

    public ObservableCollection<string> Patterns { get; } = new ObservableCollection<string>();
    public ObservableCollection<ConfigUIRawVM> Rows { get; } = new ObservableCollection<ConfigUIRawVM>();

    private string _selectedPattern;
    public string SelectedPattern
    {
        get { return _selectedPattern; }
        set
        {
            if (_selectedPattern == value) return;
            _selectedPattern = value;
            OnPropertyChanged("SelectedPattern");
            BuildRowsForPattern();
        }
    }

    public ICommand LoadProfileCommand { get; }

    public BistModeViewModel()
    {
        LoadProfileCommand = new RelayCommand(_ => LoadProfile(@"Assets/Profiles/NT51365.profile.json"));
    }

    void LoadProfile(string path)
    {
        var json = File.ReadAllText(path);
        var opts = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
        _root = JsonSerializer.Deserialize<ProfileRoot>(json, opts);

        Patterns.Clear();
        foreach (var p in _root.patterns) Patterns.Add(p.name);

        if (Patterns.Count > 0) SelectedPattern = "Red"; // 也可用 Patterns[0]
    }

    void BuildRowsForPattern()
    {
        Rows.Clear();
        if (_root == null || string.IsNullOrEmpty(SelectedPattern)) return;

        var pat = _root.patterns.FirstOrDefault(p =>
            string.Equals(p.name, SelectedPattern, StringComparison.OrdinalIgnoreCase));
        if (pat == null) return;

        foreach (var raw in pat.configUIRaws)
            Rows.Add(new ConfigUIRawVM(raw));
    }

    public event PropertyChangedEventHandler PropertyChanged;
    void OnPropertyChanged(string name)
    { var h = PropertyChanged; if (h != null) h(this, new PropertyChangedEventArgs(name)); }
}

// 每個 configUIRaw 的最小 VM（只處理 D2/D1/D0）
public class ConfigUIRawVM : INotifyPropertyChanged
{
    public string Title { get; private set; }
    public string Kind  { get; private set; }

    public IList<int> D2Options { get; private set; }
    public IList<int> D1Options { get; private set; }
    public IList<int> D0Options { get; private set; }

    private int _d2, _d1, _d0;
    public int D2 { get { return _d2; } set { _d2 = value; OnPropertyChanged("D2"); } }
    public int D1 { get { return _d1; } set { _d1 = value; OnPropertyChanged("D1"); } }
    public int D0 { get { return _d0; } set { _d0 = value; OnPropertyChanged("D0"); } }

    public ConfigUIRawVM(ConfigUIRaw raw)
    {
        Title = raw.title;
        Kind  = raw.kind;

        var d2 = raw.fields.ContainsKey("D2") ? raw.fields["D2"] : new FieldDef { min = 0, max = 3,  @default = 0 };
        var d1 = raw.fields.ContainsKey("D1") ? raw.fields["D1"] : new FieldDef { min = 0, max = 15, @default = 0 };
        var d0 = raw.fields.ContainsKey("D0") ? raw.fields["D0"] : new FieldDef { min = 0, max = 15, @default = 0 };

        D2Options = MakeRange(d2.min, d2.max);
        D1Options = MakeRange(d1.min, d1.max);
        D0Options = MakeRange(d0.min, d0.max);

        D2 = Clamp(d2.@default, d2.min, d2.max);
        D1 = Clamp(d1.@default, d1.min, d1.max);
        D0 = Clamp(d0.@default, d0.min, d0.max);
    }

    static List<int> MakeRange(int a, int b) { var list = new List<int>(); for (var i = a; i <= b; i++) list.Add(i); return list; }
    static int Clamp(int v, int lo, int hi) { if (v < lo) return lo; if (v > hi) return hi; return v; }

    public event PropertyChangedEventHandler PropertyChanged;
    void OnPropertyChanged(string name) { var h = PropertyChanged; if (h != null) h(this, new PropertyChangedEventArgs(name)); }
}

// 最小 RelayCommand
public class RelayCommand : ICommand
{
    private readonly Action<object> _exec; private readonly Func<object, bool> _can;
    public RelayCommand(Action<object> exec, Func<object, bool> can = null) { _exec = exec; _can = can; }
    public bool CanExecute(object p) { return _can == null || _can(p); }
    public void Execute(object p) { _exec(p); }
    public event EventHandler CanExecuteChanged { add { } remove { } }
}