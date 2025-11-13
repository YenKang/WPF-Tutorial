using System.Collections.Generic;
using OSDIconFlashMap.Model;

public IconOSDExportRoot BuildExportModel()
{
    var root = new IconOSDExportRoot();
    root.Ics = new List<IconOSDExportIC>();

    ICType[] icList = new ICType[]
    {
        ICType.Primary, ICType.L1, ICType.L2, ICType.L3
    };

    for (int i = 0; i < icList.Length; i++)
    {
        ICType ic = icList[i];

        // 1. 找這顆 IC 的資料
        List<IconSlotModel> slots;
        if (!_icSlotStorage.TryGetValue(ic, out slots))
        {
            slots = CreateDefaultIconSlots();
            _icSlotStorage[ic] = slots;
        }

        // 2. 準備匯出 IC 區塊
        var icExport = new IconOSDExportIC();
        icExport.Ic = ic.ToString();
        icExport.Slots = new List<IconOSDExportSlot>();

        // 3. 將每列轉成 JSON slot
        for (int s = 0; s < slots.Count; s++)
        {
            var src = slots[s];
            var dst = new IconOSDExportSlot();

            dst.IconIndex = src.IconIndex;

            if (src.SelectedImage != null)
                dst.FlashImageName = src.SelectedImage.Name;

            dst.SramStartAddress = src.SramStartAddress;
            dst.OsdIndex = src.OsdIndex;

            if (src.OsdSelectedImage != null)
                dst.OSDIconName = src.OsdSelectedImage.Name;

            dst.OsdEn = src.IsOsdEnabled;
            dst.TtaleEn = src.IsTtaleEnabled;
            dst.HPos = src.HPos;
            dst.VPos = src.VPos;

            icExport.Slots.Add(dst);
        }

        root.Ics.Add(icExport);
    }

    return root;
}