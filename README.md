public void OpenAsilIconPicker(RegAsciiSlotModel slot, Window owner)
{
    if (slot == null || owner == null)
        return;

    var osdButtons = OSDICButtonList
        .Where(b => b.OsdSelectedImage != null)
        .OrderBy(b => b.OsdIndex)
        .ToList();

    if (osdButtons.Count == 0)
    {
        MessageBox.Show(owner,
            "目前沒有任何 OSD 已選圖，請先在 Main 頁面為 OSD 選圖。",
            "提示",
            MessageBoxButton.OK,
            MessageBoxImage.Information);
        return;
    }

    var candidates = osdButtons
        .Select(b => b.OsdSelectedImage)
        .ToList();

    // ✅ 這裡只是「預選」顯示用，不會改資料
    string preKey = slot.SelectedImage?.Name;

    var picker = new ImagePickerWindow(candidates, preKey, null);
    picker.Owner = owner;

    // ✅ 只有在「按下確定」而且真的有選圖時，才會改 slot
    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 🔥 這裡才開始「覆寫」結果
        slot.SelectedImage = picker.Selected;

        int idx = candidates.IndexOf(picker.Selected);
        if (idx >= 0 && idx < osdButtons.Count)
        {
            var osdBtn = osdButtons[idx];
            slot.SelectedOsdIndex = osdBtn.OsdIndex;
        }

        Debug.WriteLine(
            $"[ASIL] Reg={slot.RegName}, " +
            $"選到圖={picker.Selected.Name}, OSD#={slot.SelectedOsdIndex}");
    }
    else
    {
        // ❗ 這個 else 裡面「不要」動 SelectedImage / SelectedOsdIndex
        Debug.WriteLine(
            $"[ASIL] Reg={slot.RegName}, 取消選圖或沒有更改，維持原狀：圖={slot.SelectedImage?.Name}, OSD#={slot.SelectedOsdIndex}");
    }
}
