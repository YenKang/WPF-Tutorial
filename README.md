using System;

/// <summary>
/// 把 Flash 的「1bpp 壓縮 bitstream」還原成「每 pixel 一格的 0/1 陣列」
///
/// ✅ 規則：
/// 1) 1 bit = 1 pixel
/// 2) MSB first：同一個 byte 內，bit7 → bit0 依序代表下一個 pixel
/// 3) row-major：畫面掃描順序是「左到右、上到下」
///
/// ------------------------------
/// 【案例】寬=3, 高=2 的圖：
///   Row0: 1 0 1
///   Row1: 1 1 1
///
/// 將 pixel 串成 bitstream： 1 0 1  1 1 1   (共 6 bits)
/// 因為 Flash 以 byte 為單位存，所以會補滿 8 bits：
///   1 0 1 1 1 1 0 0  => 二進位 10111100 => 0xBC
///
/// 所以輸入：data = { 0xBC }
/// 期待輸出：dataArr = [1,0,1, 1,1,1]
/// ------------------------------
/// </summary>
static byte[] FlashDataToByteArray(int osdWidth, int osdHeight, byte[] data)
{
    // 總 pixel 數 (例如 3x2 = 6)
    int totalPixels = osdWidth * osdHeight;

    // 輸出：每個 pixel 用 1 byte 表示，值只會是 0 或 1
    byte[] dataArr = new byte[totalPixels];

    // pixelIndex：寫入 dataArr 的位置（第幾個 pixel）
    int pixelIndex = 0;

    // bitIndex：從 bitstream 讀到第幾個 bit（第幾個 pixel）
    int bitIndex = 0;

    // 依照 row-major：y(列) 外圈，x(欄) 內圈
    for (int y = 0; y < osdHeight; y++)
    {
        for (int x = 0; x < osdWidth; x++)
        {
            // ------------------------------
            // 【bitIndex -> data[] 的定位方式】
            //
            // byteIndex = bitIndex / 8
            // 代表：第 bitIndex 個 bit 在 data[byteIndex] 裡面
            //
            // bitInByte = 7 - (bitIndex % 8)
            // 因為 MSB first：先讀 bit7，再 bit6 ... 最後 bit0
            //
            // 【案例：data = 0xBC = 10111100】
            // bitIndex=0 -> byteIndex=0, bitInByte=7 -> 取到 '1'
            // bitIndex=1 -> byteIndex=0, bitInByte=6 -> 取到 '0'
            // bitIndex=2 -> byteIndex=0, bitInByte=5 -> 取到 '1'
            // bitIndex=3 -> byteIndex=0, bitInByte=4 -> 取到 '1'
            // bitIndex=4 -> byteIndex=0, bitInByte=3 -> 取到 '1'
            // bitIndex=5 -> byteIndex=0, bitInByte=2 -> 取到 '1'
            // （後面 bitIndex=6,7 是 padding，不屬於圖的 6 個 pixel，所以不會再讀）
            // ------------------------------

            int byteIndex = bitIndex >> 3;          // bitIndex / 8
            int bitInByte = 7 - (bitIndex & 7);     // MSB first

            // 取出該 bit：
            // 1) 右移 bitInByte，把目標 bit 移到最低位
            // 2) & 1 只留最低位
            //
            // 【案例：data[0] = 0xBC = 10111100】
            // bitIndex=2 時：bitInByte=5
            // (0xBC >> 5) = 0b00000101
            // & 1 => 1
            byte pixel01 = (byte)((data[byteIndex] >> bitInByte) & 0x01);

            // 寫入輸出陣列：每 pixel 一格
            dataArr[pixelIndex++] = pixel01;

            // 前進到下一個 bit（下一個 pixel）
            bitIndex++;
        }
    }

    // 【案例最後輸出】
    // dataArr = [1,0,1, 1,1,1]
    return dataArr;
}
