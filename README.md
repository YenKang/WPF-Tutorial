// 檔案：Model/SramButton.cs
namespace OSDIconFlashMap.Model
{
    /// <summary>
    /// 對應 SRAM1、SRAM2… 的一筆記錄
    /// （順序 = 畫面上有選圖的 Icon 出現順序）
    /// </summary>
    public class SramButton
    {
        public int  SramIndex      { get; set; }  // SRAM1=1, SRAM2=2…
        public int  IconIndex      { get; set; }  // 來源的 Icon#（1~30）
        public string ImageName    { get; set; }  // image1 / 左轉向燈…
        public uint StartAddress   { get; set; }  // SRAM Start Address（數值）
        public uint ByteSize       { get; set; }  // 這張圖實際佔用的 byte 數
    }
}