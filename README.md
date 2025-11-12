using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;
using OSDIconFlashMap.View; // 你的 ImagePickerWindow 命名空間

private void OpenPicker_Click(object sender, RoutedEventArgs e)
{
    var btn  = (Button)sender;
    var slot = (IconSlotModel)btn.DataContext;
    var vm   = (IconToImageMapViewModel)this.DataContext;

    // 1) 準備圖片清單（沿用你既有做法：固定/假資料 or 掃資料夾）
    vm.LoadImagesFromFolder("Assets/TestIcons");

    // 2) 準備「已被其他列使用」清單（排除自己）
    var used = new HashSet<string>(
        vm.IconSlots.Where(s => !ReferenceEquals(s, slot))
                    .Select(s => s.SelectedImage)
                    .Where(img => img != null && !string.IsNullOrEmpty(img.Key))
                    .Select(img => img.Key)
    );

    // 3) 當前列之前選過的 key（可為 null）
    var preKey = slot.SelectedImage == null ? null : slot.SelectedImage.Key;

    // 4) 開圖片牆
    var picker = new ImagePickerWindow(vm.Images, preKey, used) { Owner = this };
    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 5) 回填
        slot.SelectedImage = picker.Selected;
        // SRAM 位址會因 SelectedImage setter → RecomputeSramStartAddress() 自動更新
    }
}