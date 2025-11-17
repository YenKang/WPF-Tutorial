## ViewMode

```
public List<int?> IconFLHSAddrList { get; private set; } = new List<int?>();

/// <summary>
/// 重建 IconFLHSAddrList
/// 排序規則：Icon_#1 → index 0, Icon_#2 → index 1 … Icon_#30 → index 29
/// 沒選圖 → 塞 null
/// </summary>
public void BuildIconFLHSAddrList()
{
    // 你已經確定 icon 是固定 1~30
    const int ICON_COUNT = 30;

    // 重新建立空白 List (全部預設 null)
    IconFLHSAddrList = new List<int?>(new int?[ICON_COUNT]);

    foreach (var slot in IconSlots)
    {
        // slot.IconNumber 必須是 1~30
        int idx = slot.IconNumber - 1;

        if (idx < 0 || idx >= ICON_COUNT)
            continue;

        // 有選圖
        if (slot.SelectedImage != null)
        {
            IconFLHSAddrList[idx] = slot.SelectedImage.FlashStartAddress;
        }
        else
        {
            // 沒選圖
            IconFLHSAddrList[idx] = null;
        }
    }

    // UI 使用到的話記得通知
    OnPropertyChanged(nameof(IconFLHSAddrList));
}
```


## 2. IconSlot

```
public class IconSlot
{
    public int IconNumber { get; set; }  // 1~30 固定編號

    public ImageOption SelectedImage { get; set; }  // 使用者選到的圖
}

public class ImageOption
{
    public string ImageKey { get; set; }
    public int FlashStartAddress { get; set; }   // 關鍵：你要用這個
}
```

## 3.Image Selection

```
private void OpenImagePicker(IconSlot slot)
{
    var win = new ImagePickerWindow(Images, slot.SelectedImage);
    if (win.ShowDialog() == true)
    {
        // 1. 回寫選到的圖
        slot.SelectedImage = win.SelectedImage;

        // 2. 更新整張表格（SRAM 你自己的流程）
        RecalculateSramAddressesByImageSize();

        // 3. 更新 IconFLHSAddrList（你現在要做的新功能）
        BuildIconFLHSAddrList();
    }
}
```