public int HRes { get => GetValue<int>(); set => SetValue(value); }
public int VRes { get => GetValue<int>(); set => SetValue(value); }

private const int HResMin = 720, HResMax = 6720;
private const int VResMin = 136, VResMax = 2560;

private static int Clamp(int v, int min, int max) => v < min ? min : (v > max ? max : v);

private void ApplyToRegister()
{
    // 這裡才做夾範圍
    HRes = Clamp(HRes, HResMin, HResMax);
    VRes = Clamp(VRes, VResMin, VResMax);

    // 之後照你原本的 Apply() 流程寫暫存器……
}

======

<TextBox Width="80"
         Text="{Binding ChessBoardVM.HRes, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
<TextBox Width="80"
         Text="{Binding ChessBoardVM.VRes, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />