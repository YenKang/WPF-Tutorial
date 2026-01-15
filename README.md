/// <summary>
/// 將 Flash 1bpp bitstream 轉成 pixel array (0/1)
///
/// Flash 中的圖片格式是：
///   - 1 bit = 1 pixel
///   - 同一個 byte 內使用 MSB first（bit7 → bit0）
///   - 像素排列順序是 row-major（左到右、上到下）
///
/// 例如 3x2 圖：
///   1 0 1
///   1 1 1
///
/// 在 Flash 會被壓成 bitstream：
///   10111100 = 0xBC
///
/// 這個函式要做的事：
///   0xBC  →  [1,0,1, 1,1,1]
/// 也就是「每一個 bit 拆成一個 byte 的 0/1」
/// </summary>
static byte[] FlashDataToByteArray(int osdWidth, int osdHeight, byte[] data)
{
    // 圖片總 pixel 數
    // 例如 3x2 = 6
    int totalPixels = osdWidth * osdHeight;

    // 這些 pixel 在 1bpp Flash 裡至少需要多少 bytes
    // 例如 6 bits → 1 byte
    int requiredBytes = (totalPixels + 7) / 8;

    if (data == null)
        throw new ArgumentNullException(nameof(data));

    if (data.Length < requiredBytes)
        throw new ArgumentException($"Flash data too short, need {requiredBytes} bytes");

    // 輸出：每一個 pixel 用一個 byte 表示 (0 或 1)
    // 長度 = osdWidth * osdHeight
    byte[] dataArr = new byte[totalPixels];

    // pixelIndex 指向 dataArr 中「第幾個 pixel」
    int pixelIndex = 0;

    // bitIndex 指向 Flash bitstream 中「第幾個 bit」
    // 0 = 第一個 pixel，1 = 第二個 pixel ...
    int bitIndex = 0;

    // 依畫面掃描順序（row-major）取 pixel
    for (int y = 0; y < osdHeight; y++)
    {
        for (int x = 0; x < osdWidth; x++)
        {
            // 這個 pixel 對應到 Flash data 的哪一個 byte
            // 例如 bitIndex = 10 → 10 / 8 = 1 → data[1]
            int byteIndex = bitIndex >> 3;

            // 在這個 byte 裡是第幾個 bit
            // 因為 Flash 是 MSB first：
            //   bitIndex%8 = 0 → bit7
            //   bitIndex%8 = 1 → bit6
            //   ...
            int bitInByte = 7 - (bitIndex & 7);

            // 取出該 bit：
            // 1. data[byteIndex] >> bitInByte 把目標 bit 移到最低位
            // 2. & 1 只留下 0 或 1
            byte pixel = (byte)((data[byteIndex] >> bitInByte) & 0x01);

            // 存到 pixel array
            dataArr[pixelIndex++] = pixel;

            // 前進到下一個 Flash bit
            bitIndex++;
        }
    }

    return dataArr;
}
