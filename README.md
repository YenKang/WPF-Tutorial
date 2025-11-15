using System;

namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// FlashButton = 就是最終要寫進 Flash Layout 的資料模型
    /// </summary>
    public class FlashButton
    {
        /// <summary>圖片檔案完整路徑，例如 C:\Icons\image1.png</summary>
        public string ImageFilePath { get; set; }

        /// <summary>決定此圖要燒錄到 Flash 的 SRAM 起始位址</summary>
        public uint FlashStartAddress { get; set; }

        /// <summary>此圖編碼成 IC 專用格式後的 Byte Size</summary>
        public uint IconSramByteSize { get; set; }
    }
}