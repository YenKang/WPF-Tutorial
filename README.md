public void OpenAsilIconPicker(RegAsciiSlotModel slot, Window owner)
{
    if (slot == null || owner == null)
        return;

    // 1️⃣ 先抓出「有選圖的」 OSD button
    var osdButtons = OSDICButtonList
        .Where(b => b.OsdSelectedImage != null)
        .OrderBy(b => b.OsdIndex)
        .Take(16)
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

    // 2️⃣ 建立圖片牆的來源（List<ImageOption>）
    var candidates = osdButtons
        .Select(b => b.OsdSelectedImage)
        .ToList();

    // 3️⃣ 預選圖名（可以為 null）
    string preKey = slot.SelectedImage?.Name;

    var picker = new ImagePickerWindow(
        candidates,
        preKey,
        null        // ASIL 不需要 usedKeys
    );
    picker.Owner = owner;

    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 4️⃣ ASIL 這一列，記住選到的這張圖
        slot.SelectedImage = picker.Selected;

        // 5️⃣ 用「圖檔名稱」去 OSDICButtonList 反查是誰在用這張圖
        var match = osdButtons
            .FirstOrDefault(b =>
                b.OsdSelectedImage != null &&
                String.Equals(
                    b.OsdSelectedImage.Name,
                    picker.Selected.Name,
                    StringComparison.OrdinalIgnoreCase));

        if (match != null)
        {
            slot.SelectedOsdIndex = match.OsdIndex;
        }
        else
        {
            slot.SelectedOsdIndex = 0; // 找不到時給預設值
        }

        // 6️⃣ Debug 印出來驗證：這一格 ASIL 選圖對應到哪個 OSD#
        Debug.WriteLine(
            $"[ASIL] Reg={slot.RegName}, " +
            $"選圖={picker.Selected.Name}, " +
            $"對應 OSD#={slot.SelectedOsdIndex}");
    }
}
