// IconToImageMapViewModel.cs

public void RecalculateSramAddressesByImageSize()
{
    // 1) 先找出「有選圖片」的列，照 IconIndex 排序
    var selectedSlots = IconSlots
        .Where(s => s.SelectedImage != null)
        .OrderBy(s => s.IconIndex)
        .ToList();

    uint runningAddr = 0;

    // 2) 依序替每一個「有選圖」的 slot 配一個 SRAM 位址
    foreach (var slot in selectedSlots)
    {
        // 2-1) 設定這一列在畫面上要顯示的 SRAM 位址
        slot.SramStartAddress = string.Format("0x{0:X6}", runningAddr);

        // 2-2) 取得這張圖片佔用的 Byte 數，往後推
        //     建議 ByteSize 放在 ImageOption 裡（你從 FlashButtonList 轉進來時一起塞）
        uint size = slot.SelectedImage.ByteSize;

        runningAddr += size;
    }

    // 3) 沒選圖的列，全部顯示 "-"
    foreach (var slot in IconSlots.Where(s => s.SelectedImage == null))
    {
        slot.SramStartAddress = "-";
    }

    // 4) 附送：Debug log，方便你在 Output 視窗看結果
    DumpSramMap();
}