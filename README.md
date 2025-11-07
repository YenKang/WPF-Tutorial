namespace OSDIconFlashMap.Model
{
    // 圖片牆的候選圖片（N 張）
    public class ImageOption
    {
        public int    Id { get; set; }          // 1..N
        public string Name { get; set; }        // 顯示名稱（檔名或自訂）
        public string Key { get; set; }         // 唯一鍵（可=Name）
        public string ThumbnailKey { get; set; }// 先留著，未來接縮圖規則
        public string ImagePath { get; set; }   // 實體檔案路徑（顯示用）
    }
}