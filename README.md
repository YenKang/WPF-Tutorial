private void OpenPicker_Click(object sender, RoutedEventArgs e)
{
    var fe = sender as FrameworkElement;
    if (fe == null) return;

    var row = fe.DataContext as IconSlotModel;
    if (row == null) return;

    var vm = (IconToImageMapViewModel)DataContext;

    // 這一列上次選的 Key（用來標黃、預選）
    var preKey = row.SelectedImage != null ? row.SelectedImage.Key : null;

    // 其他列已經使用過的 Key（用來打開圖片牆就標示）
    var usedKeys = vm.IconSlots
                     .Where(s => !object.ReferenceEquals(s, row) && s.SelectedImage != null)
                     .Select(s => s.SelectedImage.Key)
                     .ToHashSet();

    var picker = new ImagePickerWindow(vm.Images, preKey, usedKeys) { Owner = this };

    if (picker.ShowDialog() == true && picker.Selected != null)
        row.SelectedImage = picker.Selected;
}