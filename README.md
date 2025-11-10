private void OpenPicker_Click(object sender, RoutedEventArgs e)
{
    if ((sender as FrameworkElement)?.DataContext is not IconSlotModel row) return;

    var vm = (IconToImageMapViewModel)DataContext;
    var picker = new ImagePickerWindow(vm.Images) { Owner = this };

    if (picker.ShowDialog() == true)
    {
        // 寫回 Row → 觸發第 1 步的 Raise
        row.SelectedImage = picker.Selected;

        // 臨時偵錯：確認實際寫進去的是誰
        System.Diagnostics.Debug.WriteLine($"Row {row.IconIndex} => {row.SelectedImage?.Name}");
    }
}