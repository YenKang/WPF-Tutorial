private bool LoadImageFromFile(string path)
{
    if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
    {
        CentralImage = null;
        return false;
    }

    try
    {
        var bmp = new BitmapImage();
        bmp.BeginInit();
        bmp.CacheOption   = BitmapCacheOption.OnLoad;               // 讀完就關檔
        bmp.CreateOptions = BitmapCreateOptions.IgnoreImageCache;   // 同名檔會重讀
        bmp.UriSource     = new Uri(path, UriKind.Absolute);        // ✅ 用 Uri 當來源
        // 可選：bmp.DecodePixelWidth = 2560; // 你的顯示寬
        bmp.EndInit();
        bmp.Freeze();
        CentralImage = bmp;
        return true;
    }
    catch (Exception ex)
    {
        // 暫時觀察錯誤
        MessageBox.Show("LoadImageFromFile URI 版失敗：\n" + ex);
        CentralImage = null;
        return false;
    }
}