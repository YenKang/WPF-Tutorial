using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;

namespace OSDIconFlashMap.View
{
    public partial class IconToImageMapWindow : Window
    {
        /// <summary>
        /// 「Image Selection」按鈕點擊：開圖片牆
        /// 規則：
        /// 1) 圖片牆永遠顯示全部載入的圖片（不做任何篩選）
        /// 2) 對已被「其他列」使用的圖片做視覺標示，但仍允許重複選取
        /// 3) 對「本列目前選的」也做視覺標示
        /// </summary>
        private void OpenPicker_Click(object sender, RoutedEventArgs e)
        {
            var button = sender as Button;
            var slot   = button?.DataContext as IconSlotModel;
            var vm     = DataContext as IconToImageMapViewModel;
            if (vm == null || slot == null) return;

            // 確保圖片來源存在（若外部曾清空，這裡補一次）
            if (vm.Images.Count == 0)
                vm.LoadImagesFromFolder(@"Assets\TestIcons");

            // 其他列使用中的 Key（排除自己），僅供圖片牆做「標示」
            var usedByOthers = vm.IconSlots
                .Where(s => !ReferenceEquals(s, slot))
                .Select(s => s.SelectedImage?.Key)
                .Where(k => !string.IsNullOrEmpty(k))
                .ToHashSet();

            // 本列目前選取的 Key（可為 null）
            var currentKey = slot.SelectedImage?.Key;

            // 開啟圖片牆（不做過濾，永遠帶入 vm.Images 全集）
            var picker = new ImagePickerWindow(vm.Images, currentKey, usedByOthers)
            {
                Owner = this
            };

            // 允許重複選取：一律接受回傳值
            if (picker.ShowDialog() == true && picker.Selected != null)
            {
                slot.SelectedImage = picker.Selected; // 會自動觸發 SelectedImageName 與 SRAM 重算
            }
        }
    }
}