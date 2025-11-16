// IconToImageMapViewModel.cs

/// <summary>
/// 依據目前所有 IconSlots 的 Image Selection，
/// 重新計算每列的 SRAM Start Address，
/// 並依「畫面上選圖的順序」重建 SRAMButtonList。
/// </summary>
public void RecalculateSramAddressesByImageSize()
{
    // 1. 預設：全部先顯示 "-"
    foreach (var slot in IconSlots)
    {
        slot.SramStartAddress = "-";
    }

    // 2. 清空舊的 SRAMButtonList
    SramButtonList.Clear();

    // 3. 把「有選圖」的列抓出來，照 IconIndex 排序
    //    這個迴圈順序 = SRAM1, SRAM2, SRAM3… 的順序
    var selectedSlots = IconSlots
        .Where(s => s.SelectedImage != null)
        .OrderBy(s => s.IconIndex)
        .ToList();

    uint currentAddr = 0;   // 下一個 SRAM 起始位址
    int sramIndex    = 1;   // SRAM1 從 1 開始

    foreach (var slot in selectedSlots)
    {
        // 3-1. 由 SelectedImage.Key 找出該圖 byte size
        var key = slot.SelectedImage.Key;
        uint size;
        if (!_imageSizeMap.TryGetValue(key, out size))
        {
            // 找不到 size，就當 0 處理，避免 crash
            size = 0;
        }

        // 3-2. 把計算好的位址寫回欄位（給 UI 顯示）
        slot.SramStartAddress = string.Format("0x{0:X6}", currentAddr);

        // 3-3. 建立一筆 SRAMButton，存進 SRAMButtonList
        var sramBtn = new SramButton
        {
            SramIndex    = sramIndex,                 // SRAM1、SRAM2…
            IconIndex    = slot.IconIndex,           // 來源 Icon#
            ImageName    = slot.SelectedImage.Name,  // 顯示用名稱
            StartAddress = currentAddr,              // 數值型 Start Addr
            ByteSize     = size                      // 圖片 byte size
        };
        SramButtonList.Add(sramBtn);

        // 3-4. 更新下一個 SRAM 的起始位址 & index
        currentAddr += size;
        sramIndex++;
    }

    // 4. 若你有需要，可以在這裡 Debug 檢查順序
    //DumpSramMapForDebug();
}

/// <summary>
/// 開發階段用：印出 SRAMButtonList，確認順序與位址是否正確。
/// </summary>
private void DumpSramMapForDebug()
{
    System.Diagnostics.Debug.WriteLine("==== SRAM MAP DUMP ====");
    foreach (var b in SramButtonList)
    {
        System.Diagnostics.Debug.WriteLine(
            string.Format(
                "SRAM{0:D2} | Icon#{1:D2} | Img={2} | Size={3} | Addr=0x{4:X6}",
                b.SramIndex, b.IconIndex, b.ImageName, b.ByteSize, b.StartAddress));
    }
    System.Diagnostics.Debug.WriteLine("==== END SRAM MAP ====");
}