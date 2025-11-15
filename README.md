/// <summary>
/// 計算並設定第 i 顆 Icon 的「SRAM 起始位址」與「占用 Byte 數」。
/// 公式：
///   1. 先算出本顆 icon 需要多少 byte：ceil(width * height * 3 / 32)
///   2. 起始位址 = 前一顆 Icon 的 (SRAM_ADDR + SRAM_ByteSize)
/// 
/// 備註：
///   - width / height：此顆 icon 的寬、高（pixel）
///   - i：OSDButtonList 中的 index（0-based：0 = 第一顆）
/// </summary>
public virtual void SetSRAMAddr(uint width, uint height, int i)
{
    // 1️⃣ 先找出「上一顆 icon」的 SRAM 資訊
    //    - 如果 i == 0（第一顆），視為從 0 地址開始，前一顆 size 也視為 0
    uint lastSramAddr;
    uint lastSramSize;

    if (i == 0)
    {
        // 第一顆 icon：前一顆不存在，統一視為 0
        lastSramAddr = 0;
        lastSramSize = 0;
    }
    else
    {
        // 其他顆：從前一顆 OSDButton 取得已算好的 SRAM_ADDR 和 ByteSize
        lastSramAddr = OSDButtonList[i - 1].ICON_SRAM_ADDR;
        lastSramSize = OSDButtonList[i - 1].ICON_SRAM_ByteSize;
    }

    // 2️⃣ 計算本顆 icon 需要的 SRAM byte 數
    //    公式：(width * height * 3 / 32) 再做天花板 (Ceiling)
    //    - width  * height  = pixel 數
    //    - * 3 / 32         = 依照 HW 格式換算成 byte（例如 3 bits / pixel + 32bit 對齊）
    double rawSize = (double)width * (double)height * 3.0 / 32.0;
    uint byteSize  = (uint)Math.Ceiling(rawSize);

    // 寫回此顆 OSD Button 的 ByteSize
    OSDButtonList[i].ICON_SRAM_ByteSize = byteSize;

    // 3️⃣ 計算本顆 icon 的 SRAM 起始位址
    //    = 前一顆的「起始位址 + 占用大小」
    uint startAddr = lastSramAddr + lastSramSize;

    // 寫回此顆 OSD Button 的起始位址
    OSDButtonList[i].ICON_SRAM_ADDR = startAddr;
}