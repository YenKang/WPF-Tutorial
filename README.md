// FCNT1 digits
public int FCNT1_D2 { get => GetValue<int>(); set => SetValue(value); } // [9:8] 0..3
public int FCNT1_D1 { get => GetValue<int>(); set => SetValue(value); } // [7:4] 0..15
public int FCNT1_D0 { get => GetValue<int>(); set => SetValue(value); } // [3:0] 0..15

// FCNT2 digits
public int FCNT2_D2 { get => GetValue<int>(); set => SetValue(value); } // 高兩位（10~11）
public int FCNT2_D1 { get => GetValue<int>(); set => SetValue(value); }
public int FCNT2_D0 { get => GetValue<int>(); set => SetValue(value); }