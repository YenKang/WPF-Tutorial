/// <summary>
/// 將 Flash 1bpp bitstream 轉成 pixel array (每 pixel 一個 byte，值為 0 或 1)
/// bitstream 規則：
/// - 1 bit = 1 pixel
/// - MSB first（同一個 byte 內 bit7 → bit0）
/// - 依畫面順序 row-major（左到右、上到下）
/// </summary>
static byte[] FlashDataToByteArray(int osdWidth, int osdHeight, byte[] data)
{
    // 總 pixel 數
    int totalPixels = osdWidth * osdHeight;

    // 這麼多 pixel 在 1bpp 下至少需要多少 bytes
    int requiredBytes = (totalPixels + 7) / 8;

    if (data == null)
        throw new ArgumentNullException(nameof(data));

    if (data.Length < requiredBytes)
        throw new ArgumentException($"Flash data too short, need {requiredBytes} bytes");

    // 輸出：每一個 pixel 用一個 byte 表示 (0 或 1)
    byte[] dataArr = new byte[totalPixels];

    int pixelIndex = 0;   // 指向 dataArr (第幾個 pixel)
    int bitIndex = 0;     // 指向 Flash bitstream (第幾個 bit)

    // 依照畫面掃描順序：一列一列、一個 pixel 一個 pixel 取
    for (int y = 0; y < osdHeight; y++)
    {
        for (int x = 0; x < osdWidth; x++)
        {
            // 這個 pixel 對應到 Flash data 的哪一個 byte
            int byteIndex = bitIndex >> 3;          // 等同 bitIndex / 8

            // 在該 byte 裡是第幾個 bit（MSB first）
            // bit0 → bit7, bit1 → bit6 ... bit7 → bit0
            int bitInByte = 7 - (bitIndex & 7);

            // 取出該 bit，結果只會是 0 或 1
            byte pixel = (byte)((data[byteIndex] >> bitInByte) & 0x01);

            // 存到輸出的 pixel array
            dataArr[pixelIndex++] = pixel;

            // 前進到下一個 Flash bit
            bitIndex++;
        }
    }

    return dataArr;
}
