using System;
using System.Collections.ObjectModel;
using System.Collections.Generic;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;

public class BistModeViewModel : INotifyPropertyChanged
{
    public ObservableCollection<ConfigUIRaw> ConfigUiRaws { get; private set; }
    public ICommand ApplyRowCommand { get; private set; }

    public BistModeViewModel()
    {
        // 先用硬碼放一筆（等你接 JSON 後，改成載入）
        var row = new ConfigUIRaw();
        row.title  = "GrayLevel [9:0]";
        row.kind   = "grayLevelControl";
        row.fields = new Dictionary<string, FieldDef>();
        row.fields["D2"] = new FieldDef { min=0,  max=3,  @default=3  };
        row.fields["D1"] = new FieldDef { min=0,  max=15, @default=15 };
        row.fields["D0"] = new FieldDef { min=0,  max=15, @default=15 };
        row.writes = new List<WriteOp>
        {
            new WriteOp { mode="write8", target="BIST_PT_LEVEL",    src="LowByte" },
            new WriteOp { mode="rmw",    target="BIST_CK_GY_FK_PT", mask="0x03", shift=0, src="D2" }
        };
        row.explain = "GrayLevel = (D2<<8) | (D1<<4) | D0";

        ConfigUiRaws = new ObservableCollection<ConfigUIRaw>();
        ConfigUiRaws.Add(row);

        // ⑤ 建立命令：按一下要執行該列 writes
        ApplyRowCommand = new RelayCommand(p =>
        {
            var r = p as ConfigUIRaw;
            if (r != null) ExecuteWrites(r);
        });
    }

    // ③ 實做寫入流程（用 RegMap）
    private void ExecuteWrites(ConfigUIRaw row)
    {
        if (row == null || row.writes == null) return;

        // 從 UI 取 D2/D1/D0
        int d2 = row.fields.ContainsKey("D2") ? row.fields["D2"].value : 0;
        int d1 = row.fields.ContainsKey("D1") ? row.fields["D1"].value : 0;
        int d0 = row.fields.ContainsKey("D0") ? row.fields["D0"].value : 0;

        // 組 10-bit gray 與 low/high
        int  gray10 = ((d2 & 0x3) << 8) | ((d1 & 0xF) << 4) | (d0 & 0xF);
        byte low    = (byte)(gray10 & 0xFF);
        byte high   = (byte)((gray10 >> 8) & 0xFF);

        foreach (var w in row.writes)
        {
            if (w == null || string.IsNullOrEmpty(w.mode) || string.IsNullOrEmpty(w.target)) continue;

            var mode = w.mode.ToLowerInvariant();

            if (mode == "write8")
            {
                bool hi = string.Equals(w.src, "HighByte", StringComparison.OrdinalIgnoreCase);
                RegMap.Write8(w.target, hi ? high : low);
            }
            else if (mode == "rmw")
            {
                // 準備來源值
                int srcVal;
                if (string.Equals(w.src,"D2",StringComparison.OrdinalIgnoreCase)) srcVal=d2;
                else if (string.Equals(w.src,"D1",StringComparison.OrdinalIgnoreCase)) srcVal=d1;
                else if (string.Equals(w.src,"D0",StringComparison.OrdinalIgnoreCase)) srcVal=d0;
                else if (string.Equals(w.src,"HighByte",StringComparison.OrdinalIgnoreCase)) srcVal=high;
                else srcVal=low; // LowByte

                // mask/shift
                uint mask = 0xFF;
                if (!string.IsNullOrEmpty(w.mask))
                {
                    var s = w.mask.Trim();
                    mask = s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)
                        ? Convert.ToUInt32(s.Substring(2), 16)
                        : Convert.ToUInt32(s, 10);
                }
                int shift = w.shift.HasValue ? w.shift.Value : 0;

                // RMW：讀→清→置→寫
                byte cur = RegMap.Read8(w.target);
                byte res = (byte)(((uint)cur & ~(mask << shift)) |
                                  (((uint)srcVal & mask) << shift));
                RegMap.Write8(w.target, res);
            }
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string n=null)
    { var h=PropertyChanged; if (h!=null) h(this,new PropertyChangedEventArgs(n)); }
}

// ④ 簡易 RelayCommand（如果你專案裡沒有的話）
public class RelayCommand : ICommand
{
    private readonly Action<object> _exec; private readonly Func<object,bool> _can;
    public RelayCommand(Action<object> exec, Func<object,bool> can=null){ _exec=exec; _can=can; }
    public bool CanExecute(object p){ return _can==null || _can(p); }
    public void Execute(object p){ _exec(p); }
    public event EventHandler CanExecuteChanged { add{} remove{} }
}