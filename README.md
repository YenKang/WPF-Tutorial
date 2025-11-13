using System.Linq;
using System.Collections.Generic;
// ...

namespace OSDIconFlashMap.View
{
    public partial class IconToImageMapWindow : Window
    {
        // 你原本已有建構式、OpenPicker_Click (Image用)等…

        /// <summary>
        /// 點擊「OSD ICON SELECTION」欄位的按鈕，
        /// 從目前有選圖的 Icon (Icon_DL_SEL = 1) 中，開出一個圖片牆讓使用者挑。
        /// </summary>
        private void OpenOsdIconPicker_Click(object sender, RoutedEventArgs e)
        {
            // 1) 取得這顆按鈕所在那一列的 ViewModel (IconSlotModel)
            var btn = (Button)sender;
            var slot = btn.DataContext as IconSlotModel;
            if (slot == null) return;

            // 2) 取得整個視窗的 VM
            var vm = this.DataContext as IconToImageMapViewModel;
            if (vm == null) return;

            // 3) 根據目前 IconSlots，找出有選圖的那些圖當作候選
            var candidates = vm.GetOsdCandidateImages();   // 剛剛 Step2 寫的 helper

            if (candidates.Count == 0)
            {
                MessageBox.Show("目前沒有任何 Icon 選擇圖片，無法設定 OSD ICON。",
                    "提示", MessageBoxButton.OK, MessageBoxImage.Information);
                return;
            }

            // 4) 先準備「已選的 key」: 這一列目前的 OSD ICON（可以是 null）
            string preKey = slot.OsdSelectedImage == null ? null : slot.OsdSelectedImage.Key;

            // 如果你想標示「其他列 OSD 已使用的圖片」，可以做一個 usedKeys：
            // 現在先簡單全部允許重複使用，所以給空集合
            var usedKeys = new HashSet<string>();

            // 5) 開啟圖片牆（沿用你原本的 ImagePickerWindow）
            var picker = new ImagePickerWindow(candidates, preKey, usedKeys)
            {
                Owner = this
            };

            // 6) 使用者按下 OK 而且有選圖，就寫回這一列的 OsdSelectedImage
            if (picker.ShowDialog() == true && picker.Selected != null)
            {
                slot.OsdSelectedImage = picker.Selected;
                // 若未來要連動 OSD_EN 或其它 summary，在這裡再 Raise / 計算即可
            }
        }
    }
}