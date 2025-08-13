private void LoadImageFromFile(string path)
{
    if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
    {
        CentralImage = null;
        return;
    }

    try
    {
        using (var fs = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.ReadWrite))
        {
            var bmp = new BitmapImage();
            bmp.BeginInit();
            bmp.CacheOption = BitmapCacheOption.OnLoad;           // 讀到記憶體，關檔不鎖檔
            bmp.CreateOptions = BitmapCreateOptions.IgnoreImageCache; // 忽略同名快取
            bmp.StreamSource = fs;
            bmp.EndInit();
            bmp.Freeze();                                         // 跨執行緒安全
            CentralImage = bmp;
        }
    }
    catch (IOException)
    {
        // 檔案被覆蓋當下可能尚未完成寫入，稍等再重試一次
        Task.Delay(100).ContinueWith(_ =>
        {
            if (File.Exists(path))
                Application.Current.Dispatcher.Invoke(() => LoadImageFromFile(path));
        });
    }
}