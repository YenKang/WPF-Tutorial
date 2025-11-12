using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.View
{
    /// <summary>
    /// 圖片牆：
    /// - 永遠顯示「全部載入的圖片」(options)
    /// - 只做標示（本列目前選 / 其他列使用），不做篩選、不禁用
    /// - 允許重複選取
    /// </summary>
    public partial class ImagePickerWindow : Window
    {
        private readonly List<ImageOption> _all; // 全部圖片（直接吃呼叫端傳進來的全集）
        public ImageOption Selected { get; private set; } // 對話框回傳結果

        public ImagePickerWindow(IEnumerable<ImageOption> options,
                                 string currentSelectedKey,
                                 ISet<string> usedByOthersKeys)
        {
            InitializeComponent();

            _all = options?.ToList() ?? new List<ImageOption>();

            // 先清掉舊旗標（同一批 ImageOption 可能被多次用於不同列）
            foreach (var img in _all)
            {
                img.IsUsedByOthers = false;
                img.IsCurrentSelected = false;
            }

            // 標示：其他列使用中的圖片（僅視覺提示，仍允許點選）
            if (usedByOthersKeys != null && usedByOthersKeys.Count > 0)
            {
                foreach (var img in _all)
                {
                    if (!string.IsNullOrEmpty(img.Key) && usedByOthersKeys.Contains(img.Key))
                        img.IsUsedByOthers = true;
                }
            }

            // 標示：本列目前選的圖片（藍框）
            if (!string.IsNullOrEmpty(currentSelectedKey))
            {
                var hit = _all.FirstOrDefault(x => x.Key == currentSelectedKey);
                if (hit != null)
                {
                    hit.IsCurrentSelected = true;
                    Selected = hit; // 預設選擇對象（僅供 UI 高亮）
                }
            }

            // 永遠顯示「全部圖片」：不做任何 Where/Filter
            ic.ItemsSource = null;
            ic.Items.Clear();
            ic.ItemsSource = _all;
        }

        /// <summary>
        /// 點一下就選取（或你保留雙擊也可）
        /// - 允許重複選（即使該圖已被其他列使用）
        /// - 只負責回傳結果，行為管控交由外層（本需求允許重複）
        /// </summary>
        private void OnPick(object sender, MouseButtonEventArgs e)
        {
            var fe = sender as FrameworkElement;
            var op = fe?.DataContext as ImageOption;
            if (op == null) return;

            // 更新「本列目前選」視覺高亮（藍框）
            foreach (var img in _all) img.IsCurrentSelected = false;
            op.IsCurrentSelected = true;

            Selected = op;      // 對話框回傳這張
            ic.Items.Refresh(); // 立即更新外框樣式（非必要，但看起來更順）

            DialogResult = true;
            Close();
        }
    }
}