using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.View
{
    public partial class ImagePickerWindow : Window
    {
        private readonly List<ImageOption> _all;
        public ImageOption Selected { get; private set; }

        // 方便：沒有 preSelectedKey 時可用這個建構子
        public ImagePickerWindow(IEnumerable<ImageOption> options)
            : this(options, null)
        {
        }

        // 主要建構子：可帶入「上次選過的 key」
        public ImagePickerWindow(IEnumerable<ImageOption> options, string preSelectedKey)
        {
            InitializeComponent();

            _all = options != null ? options.ToList() : new List<ImageOption>();

            // 標記「上次選過」並預選
            foreach (var img in _all)
                img.IsPreviouslySelected = (preSelectedKey != null && img.Key == preSelectedKey);

            if (!string.IsNullOrEmpty(preSelectedKey))
            {
                var hit = _all.FirstOrDefault(x => x.Key == preSelectedKey);
                if (hit != null)
                {
                    hit.IsCurrentSelected = true;   // 讓本次也高亮（藍框）
                    Selected = hit;
                }
            }

            ic.ItemsSource = _all;

            // 讓 Enter 確認、Esc 取消（可選）
            PreviewKeyDown += ImagePickerWindow_PreviewKeyDown;
        }

        private void ImagePickerWindow_PreviewKeyDown(object sender, KeyEventArgs e)
        {
            if (e.Key == Key.Escape)
            {
                Selected = null;
                DialogResult = false;
                Close();
                e.Handled = true;
            }
            else if (e.Key == Key.Enter)
            {
                if (Selected != null)
                {
                    DialogResult = true;
                    Close();
                    e.Handled = true;
                }
            }
        }

        // 卡片被點擊
        private void OnPick(object sender, MouseButtonEventArgs e)
        {
            var fe = sender as FrameworkElement;
            if (fe == null) return;

            var op = fe.DataContext as ImageOption;
            if (op == null) return;

            // 清除舊的「本次選中」標記
            for (int i = 0; i < _all.Count; i++)
                _all[i].IsCurrentSelected = false;

            // 標記新的選中
            op.IsCurrentSelected = true;
            Selected = op;

            // 立即刷新樣式
            ic.Items.Refresh();

            // 直接回傳並關閉（若想改成按 OK 再關閉，就把下面兩行拿掉）
            DialogResult = true;
            Close();
        }
    }
}