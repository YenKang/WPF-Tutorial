using System.Linq;
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

            var vm = DataContext as IconToImageMapViewModel;
            if (vm == null) { vm = new IconToImageMapViewModel(); DataContext = vm; }

            if (vm.IconSlots.Count == 0) vm.InitIconSlots();
            if (vm.Images.Count == 0)    vm.LoadImagesFromFolder(@"Assets\\TestIcons");
        }

        // 【欄2】Image Selection：來源＝全集 Images
        private void OpenPicker_Click(object sender, RoutedEventArgs e)
        {
            var vm   = DataContext as IconToImageMapViewModel;
            var slot = (sender as Button)?.DataContext as IconSlotModel;
            if (vm == null || slot == null) return;

            var preKey = slot.SelectedImage?.Key;
            var used   = vm.IconSlots
                           .Where(s => !ReferenceEquals(s, slot))
                           .Select(s => s.SelectedImage?.Key)
                           .Where(k => !string.IsNullOrEmpty(k))
                           .ToHashSet();

            var picker = new ImagePickerWindow(vm.Images, preKey, used) { Owner = this };
            if (picker.ShowDialog() == true && picker.Selected != null)
                slot.SelectedImage = picker.Selected;   // 更新名稱 + SRAM 會自動處理
        }

        // 【欄4】OSD Selection：暫時也用全集 Images
        // ⚠️ 未來切換到「SRAMButtonList」時，把 vm.Images -> vm.SRAMButtonList.Select(b => b.Image)
        private void OpenOsdPicker_Click(object sender, RoutedEventArgs e)
        {
            var vm   = DataContext as IconToImageMapViewModel;
            var slot = (sender as Button)?.DataContext as IconSlotModel;
            if (vm == null || slot == null) return;

            var preKey = slot.OsdSelectedImage?.Key;
            var used   = vm.IconSlots
                           .Where(s => !ReferenceEquals(s, slot))
                           .Select(s => s.OsdSelectedImage?.Key)
                           .Where(k => !string.IsNullOrEmpty(k))
                           .ToHashSet();

            var picker = new ImagePickerWindow(vm.Images, preKey, used) { Owner = this };
            if (picker.ShowDialog() == true && picker.Selected != null)
                slot.OsdSelectedImage = picker.Selected; // 更新顯示名
        }
    }
}