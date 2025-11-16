namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// 一筆「OSD# 使用哪一張 Icon 圖」的資料
    /// 只關心 OSD 編號與 OSD Icon 名稱
    /// </summary>
    public class OSDICButton
    {
        /// <summary>
        /// OSD 編號 (1 ~ 30)
        /// </summary>
        public int OSDIndex { get; set; }

        /// <summary>
        /// OSD Icon 的名稱
        /// （來源：OSD Icon Selection 選到的圖片名稱）
        /// </summary>
        public string OsdIconName { get; set; }
    }
}