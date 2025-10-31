public static class AutoRunCache
{
    public static bool HasMemory { get; private set; }

    public static int  Total { get; private set; } = 22;
    public static int[] Orders { get; private set; } = new int[22];

    // 以 10-bit 線性值儲存 FCNT1（便於還原 D2/D1/D0）
    public static int Fcnt1 { get; private set; } = 0x03C; // 例：預設

    public static void Save(int total, IList<int> orderIdx, int fcnt1Value)
    {
        Total = total;
        if (Orders.Length != 22) Orders = new int[22];
        for (int i = 0; i < 22; i++)
            Orders[i] = (i < orderIdx.Count) ? orderIdx[i] : 0;

        Fcnt1 = fcnt1Value & 0x3FF; // 10-bit
        HasMemory = true;
    }
}