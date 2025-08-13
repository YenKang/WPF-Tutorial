public void ClickToTouch()
{
    var p = BuildPosterPath("TouchID.bmp");

    if (!File.Exists(p))
    {
        MessageBox.Show("找不到圖片：" + p);
        StopWatch();
        return;
    }

    LoadImageFromFile(p);  // 立刻顯示
    StartWatch(p);         // 之後覆蓋同名檔 → 自動刷新
}