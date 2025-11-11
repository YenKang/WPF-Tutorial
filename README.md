private void OpenPicker_Click(object sender, RoutedEventArgs e)
{
    var fe = sender as FrameworkElement;
    if (fe == null) return;

    var row = fe.DataContext as IconSlotModel;
    if (row == null) return;

    var vm = (IconToImageMapViewModel)DataContext;

    // === 在這裡放這段 ===
    var picker = new ImagePickerWindow(vm.Images) { Owner = this };
    if (picker.ShowDialog() == true)
    {
        var selected = picker.Selected;
        if (selected != null)
            row.SelectedImage = selected;
    }
}