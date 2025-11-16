namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// 一個 SRAM slot 的資訊：
    ///  - 對應哪個 Icon 列
    ///  - 用哪一張圖
    ///  - 這張圖在 SRAM 的起始位址 / 所佔 Byte 數
    /// </summary>
    public class SRAMButton
    {
        /// <summary>SRAM 編號（SRAM1 = 1, SRAM2 = 2 ...）</summary>
        public int SramIndex { get; set; }

        /// <summary>這個 SRAM slot 是由哪一列 Icon 產生 (Icon#1 → 1, Icon#2 → 2 ...)</summary>
        public int IconIndex { get; set; }

        /// <summary>圖片的 Key（對應 ImageOption.Key，例如 "image1"）</summary>
        public string ImageKey { get; set; }

        /// <summary>圖片顯示名稱（例如 "image1" 或 "左轉向燈"）</summary>
        public string ImageName { get; set; }

        /// <summary>SRAM 起始位址（實際數值，之後要寫 register 用這個）</summary>
        public uint StartAddress { get; set; }

        /// <summary>這張圖在 SRAM 佔用的 Byte 數</summary>
        public uint ByteSize { get; set; }
    }
}