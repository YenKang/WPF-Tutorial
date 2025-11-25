// 在你的 ViewModel（例如 IconToImageViewModel）裡
public void OpenAsilIconPicker(RegAsciiSlotModel slot, Window owner)
{
    if (slot == null || owner == null)
        return;

    // 1️⃣ 取得有選圖的 OSD 列表（OSDICButtonList）
    var iconButtons = OSDICButtonList
        .Where(b => b.OsdSelectedImage != null && !string.IsNullOrEmpty(b.OsdIconName))
        .OrderBy(b => b.OsdIndex)
        .Take(16)
        .ToList();

    if (!iconButtons.Any())
    {
        MessageBox.Show(owner,
            "目前沒有任何 OSD 已選圖，請先在 Main 頁面為 OSD 選圖。",
            "提示",
            MessageBoxButton.OK,
            MessageBoxImage.Information);
        return;
    }

    // 2️⃣ 建圖片候選清單
    var candidates = iconButtons
        .Select(b => b.OsdSelectedImage)
        .ToList();

    // 3️⃣ 預選 key 或圖片名稱（可為 null）
    string preKey = slot.SelectedImage?.Name;

    // 4️⃣ 開圖片選擇視窗（沿用你現有的建構子）
    var picker = new ImagePickerWindow(candidates, preKey, null);
    picker.Owner = owner;

    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 5️⃣ 更新這格 ASIL 暫存器選到的圖片
        slot.SelectedImage = picker.Selected;

        // 6️⃣ 從選到的圖片名稱去除副檔名
        string selectedBaseName = System.IO.Path.GetFileNameWithoutExtension(picker.Selected.Name);

        // 7️⃣ 組出要比對的 Key（或直接比圖片名稱也行）
        //    如果你 wish 使用 Key，可以這樣：
        var match = iconButtons
            .FirstOrDefault(b =>
                b.Key != null &&
                b.Key.EndsWith($"_{selectedBaseName}", StringComparison.OrdinalIgnoreCase)
            );

        if (match != null)
        {
            // 8️⃣ 成功找到 OSD#，更新 slot
            slot.SelectedOsdIndex = match.OsdIndex;
        }
        else
        {
            // 9️⃣ 沒找到：可設定為 0 或其他預設值
            slot.SelectedOsdIndex = 0;
        }

        // 🔟 Debug 輸出，方便你驗證
        Debug.WriteLine($"[ASIL] Reg={slot.RegName}, 選圖={picker.Selected.Name}, 對應OSD#={slot.SelectedOsdIndex}");
    }
}
