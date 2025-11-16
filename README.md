// 用 FlashButtonList 產生圖片牆要用的 ImageOption，
// 同時建立 _imageSizeMap[name] = IconSramByteSize
public void LoadImagesFromFlashButtons(IEnumerable<FlashButton> flashButtons)
{
    if (flashButtons == null)
        return;

    // 1️⃣ 每次重新載入圖片，就把 Images / _imageSizeMap 清空
    Images.Clear();
    _imageSizeMap.Clear();

    // 1-2️⃣ 先準備一個暫存 list，收集要顯示的 ImageOption
    var options = new List<ImageOption>();
    int id = 1;

    foreach (var fb in flashButtons)
    {
        // 2️⃣ 濾掉 null、空路徑、檔案不存在
        if (fb == null)
            continue;

        if (string.IsNullOrEmpty(fb.ImageFilePath))
            continue;

        if (!File.Exists(fb.ImageFilePath))
            continue;

        // 3️⃣ 從路徑取出「檔名（不含副檔名）」當作顯示名 & Key
        string nameWithoutExt = Path.GetFileNameWithoutExtension(fb.ImageFilePath);

        var opt = new ImageOption
        {
            Id           = id++,                 // 流水號 (1,2,3,…)
            Name         = nameWithoutExt,       // 顯示名稱，例如 image1
            Key          = nameWithoutExt,       // Key：先用檔名，不含副檔名
            ThumbnailKey = nameWithoutExt + "_thumbnail", // 你原本縮圖命名規則
            ImagePath    = fb.ImageFilePath      // 實際圖片路徑
        };

        options.Add(opt);

        // 4️⃣ 建立／更新 ByteSize 對照表：之後算 SRAM 都靠這個
        uint size = fb.IconSramByteSize;

        if (!_imageSizeMap.ContainsKey(nameWithoutExt))
        {
            _imageSizeMap.Add(nameWithoutExt, size);
        }
        else
        {
            // 若同一個檔名出現多次，看你要覆蓋或保留第一次
            _imageSizeMap[nameWithoutExt] = size;
        }
    }

    // 5️⃣ 呼叫你原本的 LoadImages(...)，統一更新 Images + RaisePropertyChanged
    LoadImages(options.ToArray());
}