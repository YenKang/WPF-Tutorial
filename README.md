using System;
using System.IO;
using System.Windows;
using OSDIconFlashMap.ViewModel;

namespace OSDIconFlashMap.View
{
    public partial class IconToImageMapWindow : Window
    {
        private IconToImageMapViewModel _vm;

        public IconToImageMapWindow()
        {
            InitializeComponent();

            // 建立 ViewModel
            _vm = new IconToImageMapViewModel();

            // 綁定資料上下文
            this.DataContext = _vm;

            // 載入假圖資料
            string imageFolder = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Assets", "TestIcons");
            _vm.LoadImagesFromFolder(imageFolder);
        }

        // ---- 以下保持你原本的事件（例如 OnSaveIni / OnLoadIni） ----
        private void OnSaveIni(object sender, RoutedEventArgs e)
        {
            var dlg = new Microsoft.Win32.SaveFileDialog
            {
                Filter = "INI files (*.ini)|*.ini|All files (*.*)|*.*",
                FileName = "ImageMapConfig.ini"
            };
            if (dlg.ShowDialog(this) == true)
            {
                _vm.SaveToIni(dlg.FileName);
                MessageBox.Show("Saved successfully.", "INI");
            }
        }

        private void OnLoadIni(object sender, RoutedEventArgs e)
        {
            var dlg = new Microsoft.Win32.OpenFileDialog
            {
                Filter = "INI files (*.ini)|*.ini|All files (*.*)|*.*",
                FileName = "ImageMapConfig.ini"
            };
            if (dlg.ShowDialog(this) == true)
            {
                _vm.LoadFromIni(dlg.FileName);
                MessageBox.Show("Loaded successfully.", "INI");
            }
        }

        // 若你有開啟圖片牆的事件 OnOpenPickerClick() 也放在這裡。
    }
}