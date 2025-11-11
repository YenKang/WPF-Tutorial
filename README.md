using System.Collections.Generic;
using System.Linq;
using OSDIconFlashMap.Util;

public class IconToImageMapViewModel : ViewModelBase
{
    // 你現有的屬性：IconSlots、Images… 保留

    private const string SectionName = "Primary"; // 先固定一個區段名

    public void SaveToIni(string filePath)
    {
        var dict = IniHelper.ReadAsSections(filePath);
        var section = new Dictionary<string, string>(System.StringComparer.OrdinalIgnoreCase);

        foreach (var slot in IconSlots)
        {
            var key = $"Icon{slot.IconIndex}";
            var val = slot.SelectedImage != null ? slot.SelectedImage.Key : "";
            section[key] = val;
        }

        dict[SectionName] = section;
        IniHelper.WriteSections(filePath, dict);
    }

    public void LoadFromIni(string filePath)
    {
        var dict = IniHelper.ReadAsSections(filePath);
        if (!dict.TryGetValue(SectionName, out var section)) return;

        // 注意：要先確保 Images 已載入，才能用 Key 對應
        foreach (var slot in IconSlots)
        {
            var key = $"Icon{slot.IconIndex}";
            if (section.TryGetValue(key, out var imageKey) && !string.IsNullOrEmpty(imageKey))
                slot.SelectedImage = Images.FirstOrDefault(x => x.Key == imageKey);
            else
                slot.SelectedImage = null;
        }
    }
}