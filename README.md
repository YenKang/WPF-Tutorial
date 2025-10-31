/// <summary>
/// 將目前 AutoRun 狀態儲存到快取。
/// </summary>
/// <param name="total">目前的 Pattern Total。</param>
/// <param name="orderIndices">每個 Order 的圖樣索引。</param>
/// <param name="fcnt1">目前 FCNT1 的 10-bit 整值。</param>
public static void Save(int total, IEnumerable<int> orderIndices, int fcnt1)
{
    // 1️⃣ 安全檢查
    if (total < 0)
        total = 0;

    if (orderIndices == null)
        orderIndices = Array.Empty<int>();

    // 2️⃣ 賦值（清楚分區）
    Total        = total;
    OrderIndices = orderIndices.ToArray();
    Fcnt1        = fcnt1;

    // 3️⃣ 標記記憶狀態
    HasMemory    = true;
}