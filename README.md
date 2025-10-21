// FCNT1 mapping
private string _fc1LowTarget, _fc1HighTarget;
private byte   _fc1Mask;
private int    _fc1Shift;
private int    _fc1Default;

// FCNT2 mapping
private string _fc2LowTarget, _fc2HighTarget;
private byte   _fc2Mask;
private int    _fc2Shift;
private int    _fc2Default;

======

public ObservableCollection<int> D2Options { get; } =
    new ObservableCollection<int>(Enumerable.Range(0, 4));     // 高兩位 0..3
public ObservableCollection<int> NibbleOptions { get; } =
    new ObservableCollection<int>(Enumerable.Range(0, 16));    // 0..15

=====\

// ---------- FCNT1 ----------
var f1 = autoRunControl["fcnt1"] as JObject;
if (f1 != null)
{
    _fc1Default = ParseHex((string)f1["default"], 0x03C);

    var tgt = f1["targets"] as JObject;
    if (tgt != null)
    {
        _fc1LowTarget  = (string)tgt["low"];
        _fc1HighTarget = (string)tgt["high"];
        _fc1Mask       = ParseHexByte((string)tgt["mask"], 0x03);
        _fc1Shift      = (int?)tgt["shift"] ?? 8;
    }

    // 分解 10-bit 預設到三個 digit
    FCNT1_D2 = (_fc1Default >> 8) & 0x03;
    FCNT1_D1 = (_fc1Default >> 4) & 0x0F;
    FCNT1_D0 = (_fc1Default >> 0) & 0x0F;
}

// ---------- FCNT2 ----------
var f2 = autoRunControl["fcnt2"] as JObject;
if (f2 != null)
{
    _fc2Default = ParseHex((string)f2["default"], 0x01E);

    var tgt = f2["targets"] as JObject;
    if (tgt != null)
    {
        _fc2LowTarget  = (string)tgt["low"];
        _fc2HighTarget = (string)tgt["high"];
        _fc2Mask       = ParseHexByte((string)tgt["mask"], 0x0C);
        _fc2Shift      = (int?)tgt["shift"] ?? 10;
    }

    FCNT2_D2 = (_fc2Default >> 8) & 0x03;   // 對應 11..10 兩位
    FCNT2_D1 = (_fc2Default >> 4) & 0x0F;
    FCNT2_D0 = (_fc2Default >> 0) & 0x0F;
}