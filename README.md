using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Diagnostics;
using OSDIconFlashMap.Model;
using OSDIconFlashMap.ViewModel;

namespace OSDIconFlashMap.View
{
    public partial class IconToImageMapWindow : Window
    {
        public IconToImageMapWindow()
        {
            InitializeComponent();

            // 1) 取得/建立 ViewModel；統一由這裡設定 DataContext
            var vm = this.DataContext as IconToImageMapViewModel;
            if (vm == null)
            {
                vm = new IconToImageMapViewModel();
                this.DataContext = vm;
            }

            // 2) 建 30 列（只做一次）
            if (vm.IconSlots.Count == 0)
            {
                vm.InitIconSlots();
                Debug.WriteLine($"[IconMap] IconSlots = {vm.IconSlots.Count}"); // 應為 30
            }

            // 3) 載圖片（只做一次）
            if (vm.Images.Count == 0)
            {
                vm.LoadImagesFromFolder(@"Assets\TestIcons");
                Debug.WriteLine($"[IconMap] Images = {vm.Images.Count}"); // 應 > 0
            }

            // 4) 再加一道保險：Loaded 時複檢，避免外部覆蓋 DataContext 導致空白
            this.Loaded += (s, e) => EnsureInitialized();
        }

        /// <summary>輸出結果：IconIndex → 選到的 ImageOption</summary>
        public Dictionary<int, ImageOption> ResultMap { get; private set; }

        /// <summary>保險：確保有 30 列、有圖片</summary>
        private void EnsureInitialized()
        {
            var vm = this.DataContext as IconToImageMapViewModel;
            if (vm == null)
            {
                vm = new IconToImageMapViewModel();
                this.DataContext = vm;
            }
            if (vm.IconSlots.Count == 0) vm.InitIconSlots();
            if (vm.Images.Count == 0) vm.LoadImagesFromFolder(@"Assets\TestIcons");
        }

        /// <summary>第 2 欄「Image Selection」按鈕：開啟圖片牆（全部顯示、允許重複選）</summary>
        private void OpenPicker_Click(object sender, RoutedEventArgs e)
        {
            var vm   = this.DataContext as IconToImageMapViewModel;
            var slot = (sender as Button)?.DataContext as IconSlotModel;
            if (vm == null || slot == null) return;

            if (vm.Images.Count == 0) vm.LoadImagesFromFolder(@"Assets\TestIcons");

            // 其他列使用中的 key（僅供標示；仍可重複選）
            var usedByOthers = vm.IconSlots
                                  .Where(s => !ReferenceEquals(s, slot))
                                  .Select(s => s.SelectedImage?.Key)
                                  .Where(k => !string.IsNullOrEmpty(k))
                                  .ToHashSet();

            var currentKey = slot.SelectedImage?.Key;

            var picker = new ImagePickerWindow(vm.Images, currentKey, usedByOthers)
            {
                Owner = this
            };

            if (picker.ShowDialog() == true && picker.Selected != null)
            {
                slot.SelectedImage = picker.Selected; // setter 內會更新文字與 SRAM
            }
        }

        /// <summary>OK：轉出 ResultMap，讓 ShowDialog() 的呼叫端接收</summary>
        private void ConfirmAndClose(object sender, RoutedEventArgs e)
        {
            var vm = this.DataContext as IconToImageMapViewModel;
            if (vm != null)
                ResultMap = vm.IconSlots.ToDictionary(s => s.IconIndex, s => s.SelectedImage);

            this.DialogResult = true;
            this.Close();
        }
    }
}