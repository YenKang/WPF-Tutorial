/// <summary>
/// 給 OSD ICON SELECTION 用的候選圖片清單。
/// ✅ 不去除重複：
///    例如 Icon#1=image2, Icon#2=image1, Icon#3=image1
///    回傳順序就是 [image2, image1, image1]。
/// </summary>
public List<ImageOption> GetOsdCandidateImages()
{
    // 1. 找出所有「有選 Image」的 Icon 列
    var hasImageSlots = IconSlots
        .Where(slot => slot.SelectedImage != null)
        .OrderBy(slot => slot.IconIndex);   // 依 Icon 編號排序，方便你對照

    // 2. 直接把 SelectedImage 丟出來
    //    不做 Distinct / 不做 GroupBy
    var list = hasImageSlots
        .Select(slot => slot.SelectedImage)
        .Where(img => img != null)
        .ToList();

    return list;
}