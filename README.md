```
private void OpenAsilIconPicker_Click(object sender, RoutedEventArgs e)
{
    var btn  = sender as Button;
    var slot = btn?.DataContext as RegAsciiSlotModel;
    if (slot == null)
        return;

    if (DataContext is IconToImageViewModel vm)
    {
        vm.OpenAsilIconPicker(slot, this);   // this 當 Owner 傳進去
    }
}
```

## 6-2. 在 ViewModel 實作 OpenAsilIconPicker
```
public void OpenAsilIconPicker(RegAsciiSlotModel slot, Window owner)
{
    if (slot == null)
        return;

    // 1️⃣ 只取有選圖的 OSD，依 OSD# 排序，取前 16 個
    var iconButtons = OSDICButtonList
        .Where(b => b.OsdSelectedImage != null)
        .OrderBy(b => b.OsdIndex)
        .Take(16)
        .ToList();

    if (iconButtons.Count == 0)
    {
        MessageBox.Show(owner,
            "目前沒有任何 OSD 選圖可用，請先在 Main 頁選好 OSD Icon。",
            "提示",
            MessageBoxButton.OK,
            MessageBoxImage.Information);
        return;
    }

    // 2️⃣ 只把圖片丟給圖片牆（如果你現有的 ImagePickerWindow 是吃 List<ImageOption>）
    var candidates = iconButtons
        .Select(b => b.OsdSelectedImage)
        .ToList();

    var picker = new ImagePickerWindow(candidates, slot.SelectedImage);
    picker.Owner = owner;

    if (picker.ShowDialog() == true && picker.SelectedImage != null)
    {
        // 3️⃣ ASIL slot 記住這張圖
        slot.SelectedImage = picker.SelectedImage;

        // 4️⃣ 反查這張圖是來自哪一個 OSD（用 Equals 或 Key）
        var match = iconButtons
            .FirstOrDefault(b => b.OsdSelectedImage == picker.SelectedImage);

        if (match != null)
        {
            slot.SelectedOsdIndex = match.OsdIndex;  // 🔥 這裡就是你要的 OSD#
        }
        else
        {
            slot.SelectedOsdIndex = 0; // 或者給個預設值
        }
    }
}
```
