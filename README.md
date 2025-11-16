public IconOSDExportRoot BuildExportModel()
{
    // 1. 建立根節點
    var root = new IconOSDExportRoot();
    root.Ics = new List<IconOSDExportIC>();

    // 2. IC 列表
    ICType[] icList = new ICType[]
    {
        ICType.Primary, ICType.L1, ICType.L2, ICType.L3
    };

    for (int i = 0; i < icList.Length; i++)
    {
        ICType ic = icList[i];

        // 2-1. 找出這顆 IC 的 slot（若未建立 → 建一份預設）
        List<IconSlotModel> slots;
        if (!_icSlotStorage.TryGetValue(ic, out slots))
        {
            slots = CreateDefaultIconSlots();
            _icSlotStorage[ic] = slots;
        }

        // 2-2. 建立 IC 區塊
        var icExport = new IconOSDExportIC();
        icExport.Ic    = ic.ToString();
        icExport.Icons = new List<ExportIcon>();
        icExport.Osds  = new List<ExportOsd>();

        // 3. 每一列 slot → 塞入 Icons[] & Osds[]
        for (int s = 0; s < slots.Count; s++)
        {
            var src = slots[s];

            // ----- Icons[] 部分 -----
            var iconItem = new ExportIcon();
            iconItem.IconIndex        = src.IconIndex;
            iconItem.FlashImageName   = NormalizeNameOrNull(src.SelectedImageName);
            iconItem.SramStartAddress = src.SramStartAddress;

            icExport.Icons.Add(iconItem);

            // ----- Osds[] 部分 -----
            var osdItem = new ExportOsd();
            osdItem.OsdIndex    = src.OsdIndex;
            osdItem.OsdIconName = NormalizeNameOrNull(src.OsdSelectedImageName);
            osdItem.OsdEn       = src.IsOsdEnabled;
            osdItem.TtaleEn     = src.IsTtaleEnabled;
            osdItem.HPos        = src.HPos;
            osdItem.VPos        = src.VPos;

            icExport.Osds.Add(osdItem);
        }

        // 加入 root
        root.Ics.Add(icExport);
    }

    return root;
}

/// <summary>
/// 把 "未選擇" 改成 null
/// </summary>
private static string NormalizeNameOrNull(string name)
{
    if (string.IsNullOrEmpty(name)) return null;
    if (name == "未選擇") return null;
    return name;
}