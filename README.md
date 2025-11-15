private void OpenImagePicker_Click(object sender, RoutedEventArgs e)
{
    var btn  = (Button)sender;
    var slot = btn.DataContext as IconSlotModel;
    if (slot == null) return;

    var vm = this.DataContext as IconToImageMapViewModel;
    if (vm == null) return;

    // 這裡只是「讀取」 vm.Images，不是「載入」圖片
    var candidates = vm.Images;

    if (candidates == null || candidates.Count == 0)
    {
        MessageBox.Show("目前沒有可選擇的圖片。");
        return;
    }

    string preKey = null;
    if (slot.SelectedImage != null)
        preKey = slot.SelectedImage.Key;

    var usedKeys = new HashSet<string>(); // 或你原本用的

    var picker = new ImagePickerWindow(candidates, preKey, usedKeys);
    picker.Owner = this;

    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        slot.SelectedImage = picker.Selected;
    }
}