using OSDIconFlashMap.Model;
using System.Collections.Generic;
using System.Linq;

// ...

public class IconToImageMapViewModel : ViewModelBase
{
    // 已有：IconSlots, Images, _imageSizeMap ...

    // 1️⃣ SRAMButton 的集合，之後要給 OSD 選圖用
    private readonly List<SRAMButton> _sramButtonList = new List<SRAMButton>();

    /// <summary>
    /// 目前依照「有選圖的 Icon」所產生的 SRAM slot 列表。
    /// 排序：照 IconIndex 由小到大 = SRAM1, SRAM2, SRAM3...
    /// </summary>
    public List<SRAMButton> SramButtonList
    {
        get { return _sramButtonList; }
    }

    // ... 其他程式碼 (IconSlots, Images, LoadImagesFromFlashButtons 等)
}