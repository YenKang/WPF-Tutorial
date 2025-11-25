if (picker.ShowDialog() == true && picker.Selected != null)
{
    // ASIL slot 紀錄選到的圖片
    slot.SelectedImage = picker.Selected;

    // 從 OSDICButtonList 找這張圖是來自哪個 OSD
    var match = vm.OSDICButtonList
        .FirstOrDefault(b =>
            b.OsdSelectedImage != null &&
            b.OsdSelectedImage.Name == picker.Selected.Name);

    if (match != null)
    {
        slot.SelectedOsdIndex = match.OsdIndex;

        // ⭐ Debug — 印出 ASIL 這格的結果
        Debug.WriteLine($"[ASIL] Reg={slot.RegName}, 選到圖={picker.Selected.Name}, 對應 OSD#={match.OsdIndex}");
    }
    else
    {
        slot.SelectedOsdIndex = 0;

        // ⭐ Debug — 找不到 OSD 情況
        Debug.WriteLine($"[ASIL] Reg={slot.RegName}, 選到圖={picker.Selected.Name}, 找不到對應 OSD");
    }
}
