using Utility.MVVM;

namespace OSDIconFlashMap.Model
{
    public class OsdModel : ViewModelBase
    {
        public string OsdName { get; set; }   // "OSD_1".."OSD_40"

        private IconModel _selectedIcon;
        public IconModel SelectedIcon
        {
            get => _selectedIcon;
            set
            {
                if (ReferenceEquals(_selectedIcon, value)) return;
                _selectedIcon = value;

                // 1) 本身屬性
                RaisePropertyChanged(nameof(SelectedIcon));
                // 2) 衍生屬性（畫面綁這兩個 → 會即時刷新）
                RaisePropertyChanged(nameof(ThumbnailKey));
                RaisePropertyChanged(nameof(FlashStartAddrHex));
            }
        }

        // 綁這兩個，不要再綁 SelectedIcon.*
        public string ThumbnailKey       => SelectedIcon?.ThumbnailKey  ?? "(待載入)";
        public string FlashStartAddrHex  => SelectedIcon?.FlashStartHex ?? "0x000000";
    }
}