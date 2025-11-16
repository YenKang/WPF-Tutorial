// IconToImageMapViewModel.cs

using System.Collections.Generic;
using System.Linq;

public class IconToImageMapViewModel : ViewModelBase
{
    public ObservableCollection<IconSlotModel> IconSlots { get; private set; }

    // 你要的 SRAMButtonList
    public List<SramButton> SramButtonList { get; private set; }

    // 圖片名 → ByteSize 的 map（之前已建立）
    private readonly Dictionary<string, uint> _imageSizeMap = new Dictionary<string, uint>();

    public IconToImageMapViewModel()
    {
        IconSlots      = new ObservableCollection<IconSlotModel>();
        SramButtonList = new List<SramButton>();
    }

    ...
}