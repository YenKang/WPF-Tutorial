/// <summary>
/// 根據 FlashButtonList 產生圖片牆要用的 ImageOption 清單，
/// 等於是「把 flashButtonList[i].ImageFilePath 全部收集起來」
/// 轉成 Images（ObservableCollection&lt;ImageOption&gt;）。
/// </summary>
public void LoadImagesFromFlashButtons(IEnumerable<FlashButton> flashButtons)
{
    if (flashButtons == null)
        return;

    // 1️⃣ 先準備一個暫存 list，收集要顯示的 ImageOption
    var options = new List<ImageOption>();
    int id = 1;

    foreach (var fb in flashButtons)
    {
        // 忽略掉沒有路徑或檔案不存在的
        if (fb == null)
            continue;

        if (string.IsNullOrEmpty(fb.ImageFilePath))
            continue;

        if (!File.Exists(fb.ImageFilePath))
            continue;

        // 從路徑取出 檔名（不含副檔名），當作顯示名稱＆Key
        string nameWithoutExt = Path.GetFileNameWithoutExtension(fb.ImageFilePath);

        var opt = new ImageOption();
        opt.Id   = id++;                  // 流水號（1,2,3,...）
        opt.Name = nameWithoutExt;        // 顯示名稱，例如 image1
        opt.Key  = nameWithoutExt;        // Key：先用檔名，之後要改也容易
        opt.ThumbnailKey = nameWithoutExt + "_thumbnail"; // 你之前的縮圖命名規則
        opt.ImagePath    = fb.ImageFilePath;              // 真正圖片路徑

        options.Add(opt);
    }

    // 2️⃣ 呼叫你原本的 LoadImages(...)，統一更新 Images
    //    這樣不用重複寫「清空、Add、RaisePropertyChanged」等邏輯
    LoadImages(options.ToArray());
}