var p = BuildPosterPath("TouchID.bmp");
if (!File.Exists(p))
{
    MessageBox.Show("找不到圖片：" + p);
}
else
{
    LoadImageFromFile(p);
    MessageBox.Show("已載入：" + p);
}