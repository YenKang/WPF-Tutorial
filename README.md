public void LoadImagesFromFolder(string relativeFolder)
{
    // 以執行檔資料夾為基準
    var baseDir = AppDomain.CurrentDomain.BaseDirectory;
    var folder  = Path.Combine(baseDir, relativeFolder);

    if (!Directory.Exists(folder))
        return;

    // ✅ 1. 只根據「副檔名」找圖，不管檔名長怎樣
    var filePatterns = new[] { "*.png", "*.jpg", "*.jpeg", "*.bmp" };

    var files = filePatterns
        .SelectMany(p => Directory.EnumerateFiles(folder, p, SearchOption.TopDirectoryOnly))
        // ✅ 2. 依檔名排序，讓結果穩定（例如 image1, image2, …）
        .OrderBy(path => path, StringComparer.OrdinalIgnoreCase)
        .ToList();

    // ✅ 3. 把檔案轉成 ImageOption 清單
    var list = new List<ImageOption>();
    int id = 1;

    foreach (var path in files)
    {
        var nameWithoutExt = Path.GetFileNameWithoutExtension(path);

        list.Add(new ImageOption
        {
            Id   = id++,
            Name = nameWithoutExt,           // 顯示名稱（你可視需要調整）
            Key  = nameWithoutExt,           // Key：這邊先用檔名（不含副檔名）

            // 若你有 thumbnail 規則，可照之前約定：
            ThumbnailKey = $"{nameWithoutExt}_thumbnail",

            ImagePath = path                 // 圖片實際路徑
        });
    }

    // ✅ 4. 丟給原本的 LoadImages()，統一更新 Images
    LoadImages(list.ToArray());
}