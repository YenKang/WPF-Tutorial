using System.ComponentModel;
using OSDIconFlashMap.Model;

public class OsdModel : INotifyPropertyChanged
{
    public string OsdName { get; set; }

    private IconModel _selectedIcon;
    public IconModel SelectedIcon
    {
        get => _selectedIcon;
        set
        {
            if (_selectedIcon == value) return;
            _selectedIcon = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(SelectedIcon)));
            // 衍生屬性也要通知，讓畫面即時聯動
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ThumbnailKey)));
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(FlashAddressHex)));
        }
    }

    // 讓 XAML 綁這兩個，就會跟著 SelectedIcon 動
    public string ThumbnailKey   => SelectedIcon?.ThumbnailKey   ?? "(待載入)";
    public string FlashAddressHex=> SelectedIcon?.FlashStartHex  ?? "0x000000";

    public event PropertyChangedEventHandler PropertyChanged;
}