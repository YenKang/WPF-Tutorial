public List<ImageOption> GetOsdCandidateImages()
{
    // 1. 找出所有有選圖片的 Icon 列，依 Icon# 排序
    var hasImageSlots = IconSlots
        .Where(slot => slot.SelectedImage != null)
        .OrderBy(slot => slot.IconIndex);

    // 2. 為每一列產生一個 OsdIconCandidate
    var list = hasImageSlots
        .Select(slot =>
        {
            var img = slot.SelectedImage;
            if (img == null) return null;

            // 建一個「新的」候選物件，裡面記下 Icon#
            var cand = new OsdIconCandidate
            {
                SourceIconIndex = slot.IconIndex, // ✅ 關鍵：記住 1、4、5...

                // 下面把你需要的外觀相關屬性複製過來：
                Key         = img.Key,
                DisplayName = img.DisplayName,
                // 若你有 Bitmap 之類的，也一起複製：
                // ImageSource = img.ImageSource,
                // Thumbnail   = img.Thumbnail,
            };

            return cand;
        })
        .Where(c => c != null)
        .Cast<ImageOption>()   // 回傳型別仍是 List<ImageOption>，不用改呼叫點
        .ToList();

    return list;
}