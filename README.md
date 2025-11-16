public class IconToImageMapViewModel : ViewModelBase
{
    // 你原本的欄位...

    /// <summary>
    /// 依 OSD#1~30 排好的「OSD → Icon」清單
    /// 來源是畫面右側 OSD Icon Selection
    /// </summary>
    public List<OSDICButton> OSDICButtonList { get; private set; }

    public IconToImageMapViewModel()
    {
        IconSlots = new ObservableCollection<IconSlotModel>();
        OSDICButtonList = new List<OSDICButton>();
    }

    // ...
}