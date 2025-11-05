using System.Windows.Media.Imaging;

namespace OSDIconFlashMap.Model
{
    public class IconModel
    {
        public int Id { get; set; }                 // 1..N -> icon_1..icon_N
        public string Name { get; set; }            // "icon_1"
        public ulong FlashStart { get; set; }       // 數值位址
        public string FlashStartHex => $"0x{FlashStart:X}";
        public BitmapImage Thumbnail { get; set; }  // 之後可載縮圖
    }
}

====
using System.ComponentModel;

namespace OSDIconFlashMap.Model
{
    public class OsdModel : INotifyPropertyChanged
    {
        public string OsdName { get; set; }  // "OSD_1".."OSD_40"

        private IconModel _selectedIcon;
        public IconModel SelectedIcon
        {
            get => _selectedIcon;
            set
            {
                if (_selectedIcon != value)
                {
                    _selectedIcon = value;
                    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(SelectedIcon)));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;
    }
}