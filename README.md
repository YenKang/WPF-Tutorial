using Utility.MVVM;

namespace OSDIconFlashMap.Model
{
    public class OsdModel : ViewModelBase
    {
        // "OSD_1" ~ "OSD_40"
        public string OsdName { get; set; }

        // 綁 ICON 下拉的選項
        public IconModel SelectedIcon
        {
            get => GetValue<IconModel>();
            set
            {
                if (SetValue(value))
                {
                    // 當 ICON 改變，手動通知衍生屬性刷新
                    Raise(nameof(ThumbnailKey));
                    Raise(nameof(FlashStartAddrHex));
                }
            }
        }

        // 綁定到畫面用的衍生屬性（隨 SelectedIcon 改變）
        public string ThumbnailKey      => SelectedIcon?.ThumbnailKey   ?? "(待載入)";
        public string FlashStartAddrHex => SelectedIcon?.FlashStartHex  ?? "0x000000";

        // 封裝通知方法（依你的 ViewModelBase 版本調整）
        private void Raise(string name)
        {
            RaisePropertyChanged(name); // 若你的 base 類叫 OnPropertyChanged(name)，就換成那個
        }
    }
}