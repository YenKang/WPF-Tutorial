public static void Clear()
    {
        HasMemory = false;             // 標記無快取
        Total = 0;                     // 清掉圖片總數
        OrderIndices = new int[0];     // 清掉每張圖的順序紀錄
        Fcnt1 = 0;                     // 清掉 flicker counter1
        Fcnt2 = 0;                     // 清掉 flicker counter2

        System.Diagnostics.Debug.WriteLine("[AutoRunCache] 已清