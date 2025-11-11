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

        public ImagePickerWindow(IEnumerable<ImageOption> options)
        {
            InitializeComponent();

            // 避免 null
            _all = options != null ? options.ToList() : new List<ImageOption>();

            // 綁定資料
            ic.ItemsSource = _all;
        }

        // 點選圖片卡片
        private void OnPick(object sender, MouseButtonEventArgs e)
        {
            var fe = sender as FrameworkElement;
            if (fe == null)
                return;

            var op = fe.DataContext as ImageOption;
            if (op == null)
                return;

            // 清除舊選取
            foreach (var img in _all)
                img.IsCurrentSelected = false;

            // 標記新的選中
            op.IsCurrentSelected = true;
            Selected = op;

            // 通知 UI 重新整理
            ic.Items.Refresh();

            // 關閉視窗並回傳結果
            DialogResult = true;
            Close();
        }
    }
}