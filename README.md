using System.Windows;
using System.Windows.Controls;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;

namespace OSDIconFlashMap.View
{
    public partial class IconToImageMapWindow : Window
    {
        public IconToImageMapWindow()
        {
            InitializeComponent();

            // 若設計器報 XDG000，可改成：
            // DataContext = new IconToImageMapViewModel();

            var vm = (IconToImageMapViewModel)DataContext;
            vm.InitIconSlots();
            vm.LoadImagesFromFolder(@"Assets\TestIcons"); // 7 張測試圖
        }

        private void OpenPicker_Click(object sender, RoutedEventArgs e)
        {
            if ((sender as FrameworkElement)?.DataContext is not IconSlotModel row) return;

            var vm = (IconToImageMapViewModel)DataContext;
            var picker = new ImagePickerWindow(vm.Images) { Owner = this };

            if (picker.ShowDialog() == true)
                row.SelectedImage = picker.Selected;
        }
    }
}