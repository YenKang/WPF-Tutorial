// ====== 解析度邊界 ======
private const int HResMin = 720;
private const int HResMax = 6720;
private const int VResMin = 136;
private const int VResMax = 2560;

public int HRes
{
    get => GetValue<int>();
    set => SetValue(Clamp(value, HResMin, HResMax));
}

public int VRes
{
    get => GetValue<int>();
    set => SetValue(Clamp(value, VResMin, VResMax));
}

private static int Clamp(int v, int min, int max) => v < min ? min : (v > max ? max : v);