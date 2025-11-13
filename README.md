using System.Collections.Generic;

namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// JSON 最外層：包含所有 IC 的資料
    /// </summary>
    public class IconOSDExportRoot
    {
        /// <summary>
        /// 四種 IC 的集合：Primary / L1 / L2 / L3
        /// </summary>
        public List<IconOSDExportIC> Ics { get; set; }
    }

    /// <summary>
    /// 每一顆 IC 的資料包
    /// </summary>
    public class IconOSDExportIC
    {
        /// <summary>
        /// IC 名稱，例如 "Primary"、"L1"、"L2"、"L3"
        /// </summary>
        public string Ic { get; set; }

        /// <summary>
        /// 該 IC 底下 30 筆 Icon slots 的設定
        /// </summary>
        public List<IconOSDExportSlot> Slots { get; set; }
    }

    /// <summary>
    /// 對應 DataGrid 中一列資料（Icon #1 ~ #30）
    /// </summary>
    public class IconOSDExportSlot
    {
        /// <summary>Icon 編號（1~30）</summary>
        public int IconIndex { get; set; }

        /// <summary>FLASH 內使用的圖片名稱，例如 image1</summary>
        public string FlashImageName { get; set; }

        /// <summary>SRAM Start Address，例如 "0x0000"</summary>
        public string SramStartAddress { get; set; }

        /// <summary>OSD # 欄位</summary>
        public int OsdIndex { get; set; }

        /// <summary>OSD ICON SELECTION 選到的圖片名稱</summary>
        public string OsdIconName { get; set; }

        /// <summary>OSDx_EN</summary>
        public bool OsdEn { get; set; }

        /// <summary>TTALEEN</summary>
        public bool TtaleEn { get; set; }

        /// <summary>水平位置</summary>
        public int HPos { get; set; }

        /// <summary>垂直位置</summary>
        public int VPos { get; set; }
    }
}