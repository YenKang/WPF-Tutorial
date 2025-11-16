using System.Collections.Generic;

namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// JSON 根節點：
    /// {
    ///   "ics": [ ... ]
    /// }
    /// </summary>
    public class IconOSDExportRoot
    {
        public List<IconOSDExportIC> Ics { get; set; }

        public IconOSDExportRoot()
        {
            Ics = new List<IconOSDExportIC>();
        }
    }

    /// <summary>
    /// 每一個 IC：
    /// {
    ///   "ic": "Primary",
    ///   "icons": [...],
    ///   "osds": [...]
    /// }
    /// </summary>
    public class IconOSDExportIC
    {
        public string Ic { get; set; }

        /// <summary>Flash ICON 區</summary>
        public List<ExportIcon> Icons { get; set; }

        /// <summary>OSD 區</summary>
        public List<ExportOsd> Osds { get; set; }

        public IconOSDExportIC()
        {
            Icons = new List<ExportIcon>();
            Osds  = new List<ExportOsd>();
        }
    }

    /// <summary>
    /// Icons[]：Flash/SRAM ICON 資料
    /// </summary>
    public class ExportIcon
    {
        public int    IconIndex        { get; set; }
        public string FlashImageName   { get; set; }   // 可能為 null
        public string SramStartAddress { get; set; }   // "0x000000" or "-"
    }

    /// <summary>
    /// Osds[]：OSD 設定資料
    /// </summary>
    public class ExportOsd
    {
        public int    OsdIndex    { get; set; }
        public string OsdIconName { get; set; }   // 可能為 null

        public bool OsdEn   { get; set; }
        public bool TtaleEn { get; set; }

        public int HPos { get; set; }
        public int VPos { get; set; }
    }
}