// === Cursor Position ===
public bool ShowHPos { get => GetValue<bool>(); private set => SetValue(value); }
public bool ShowVPos { get => GetValue<bool>(); private set => SetValue(value); }

public int HPos { get => GetValue<int>(); set => SetValue(value); }
public int VPos { get => GetValue<int>(); set => SetValue(value); }

private int _hMin, _hMax, _vMin, _vMax;
private string _hLowTarget, _hHighTarget, _vLowTarget, _vHighTarget;
private byte _hMask, _vMask;