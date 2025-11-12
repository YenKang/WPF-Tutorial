using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.View
{
    /// <summary>
    /// 圖片牆：顯示全部圖片，僅做「其他列使用中 / 本列目前選」的視覺標示；
    /// 不禁用、允許重複選取。
    /// </summary>
    public partial class ImagePickerWindow : Window
    {
        private readonly List<ImageOption> _all;   // 全部圖片來源
        public ImageOption Selected { get; private set; } // 回傳選中的圖片

        public ImagePickerWindow(IEnumerable<ImageOption> options,
                                 string currentSelectedKey,
                                 ISet<string> usedByOthersKeys)
        {
            InitializeComponent();

            _all = options?.ToList() ?? new List<ImageOption>();

            // 先清旗標（同一批物件可能多次被用來開視窗）
            foreach (var img in _all)
            {
                img.IsUsedByOthers = false;
                img.IsCurrentSelected = false;
            }

            // 標示：其他列使用中
            if (usedByOthersKeys != null && usedByOthersKeys.Count > 0)
            {
                foreach (var img in _all)
                {
                    if (!string.IsNullOrEmpty(img.Key) && usedByOthersKeys.Contains(img.Key))
                        img.IsUsedByOthers = true;
                }
            }

            // 標示：本列目前選
            if (!string.IsNullOrEmpty(currentSelectedKey))
            {
                var hit = _all.FirstOrDefault(x => x.Key == currentSelectedKey);
                if (hit != null) { hit.IsCurrentSelected = true; Selected = hit; }
            }

            // 不做任何過濾：永遠顯示全部圖片
            ic.ItemsSource = null;
            ic.Items.Clear();
            ic.ItemsSource = _all;
        }

        // 點一下就選；允許重複選
        private void OnPick(object sender, MouseButtonEventArgs e)
        {
            var fe = sender as FrameworkElement;
            var op = fe?.DataContext as ImageOption;
            if (op == null) return;

            foreach (var img in _all) img.IsCurrentSelected = false;
            op.IsCurrentSelected = true;
            Selected = op;

            ic.Items.Refresh();  // 讓藍框即時更新
            DialogResult = true; // 對話框成功
            Close();
        }
    }
}