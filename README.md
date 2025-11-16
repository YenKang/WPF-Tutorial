// IconSlotModel.cs

public class IconSlotModel : ViewModelBase
{
    // 讓 Slot 知道「誰是它的 ViewModel」
    public IconToImageMapViewModel OwnerViewModel { get; set; }

    // ... 你原本的 IconIndex / SelectedImage / HPos / VPos 等等 ...
}