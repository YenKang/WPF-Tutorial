private void OnIconSlotPropertyChanged(object sender, PropertyChangedEventArgs e)
{
    // 1）Image Selection 變化 → 重算 SRAM
    if (e.PropertyName == nameof(IconSlotModel.SelectedImage))
    {
        RecalculateSramAddressesByImageSize();
    }

    // 2）OSD Icon Selection 變化 → 重建 OSDICButtonList
    if (e.PropertyName == nameof(IconSlotModel.OsdSelectedImage))
    {
        RebuildOsdIcButtonList();
    }
}

=======

/// <summary>
/// 重新統計目前畫面上「每一個 OSD# 用哪一張 Icon」
/// 1. 清空 OSDICButtonList
/// 2. 依 OSDIndex 1→30 排序 IconSlots
/// 3. 只處理有選 OSD Icon 的列
/// </summary>
public void RebuildOsdIcButtonList()
{
    if (OSDICButtonList == null)
        OSDICButtonList = new List<OSDICButton>();
    else
        OSDICButtonList.Clear();

    // 依 OSD#1~30 排序（右側 OSD # 欄位）
    var orderedSlots = IconSlots
        .OrderBy(s => s.OsdIndex)
        .ToList();

    foreach (var slot in orderedSlots)
    {
        if (slot.OsdSelectedImage == null)
            continue;   // 沒選 OSD Icon 的 OSD# 就略過

        var item = new OSDICButton
        {
            OSDIndex    = slot.OsdIndex,
            OsdIconName = slot.OsdSelectedImage.Name
        };

        OSDICButtonList.Add(item);
    }

    // Debug 輸出，方便你在 Output 視窗確認
    System.Diagnostics.Debug.WriteLine("==== OSDICButtonList ====");
    foreach (var b in OSDICButtonList)
    {
        System.Diagnostics.Debug.WriteLine(
            $"OSD#{b.OSDIndex:00} -> {b.OsdIconName}");
    }
    System.Diagnostics.Debug.WriteLine("==== END ====");
}