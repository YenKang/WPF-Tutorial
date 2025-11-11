// View/IconToImageMapWindow.xaml.cs 內的 OpenPicker_Click
private void OpenPicker_Click(object sender, RoutedEventArgs e)
{
    var fe  = sender as FrameworkElement;
    var row = fe?.DataContext as IconSlotModel;
    if (row == null) return;

    var vm = (IconToImageMapViewModel)DataContext;

    var preKey  = row.SelectedImage != null ? row.SelectedImage.Key : null;
    var usedKeys = vm.IconSlots
                     .Where(s => !ReferenceEquals(s, row) && s.SelectedImage != null)
                     .Select(s => s.SelectedImage.Key)
                     .ToHashSet(); // C# 7.3 OK (需 using System.Linq;)

    var picker = new ImagePickerWindow(vm.Images, preKey, usedKeys) { Owner = this };
    if (picker.ShowDialog() == true && picker.Selected != null)
        row.SelectedImage = picker.Selected;
}