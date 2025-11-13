using System.Collections.Generic;

namespace OSDIconFlashMap.Model
{
    // JSON root
    public class IconOSDExportRoot
    {
        public List<IconOSDExportIC> Ics { get; set; }
    }

    // Each IC block
    public class IconOSDExportIC
    {
        public string Ic { get; set; }
        public List<IconOSDExportSlot> Slots { get; set; }
    }

    // One row (Icon#1 ~ Icon#30)
    public class IconOSDExportSlot
    {
        public int IconIndex { get; set; }
        public string FlashImageName { get; set; }
        public string SramStartAddress { get; set; }
        public int OsdIndex { get; set; }
        public string OSDIconName { get; set; }
        public bool OsdEn { get; set; }
        public bool TtaleEn { get; set; }
        public int HPos { get; set; }
        public int VPos { get; set; }
    }
}