// Model/IconModel.cs
using System.Drawing;                 // 可選：如果你會帶 RawBitmap
using System.Windows.Media;           // ImageSource
// using System.Windows.Media.Imaging; // 若你也會用 BitmapImage 可保留

namespace OSDIconFlashMap.Model
{
    public class IconModel
    {
        public int Id { get; set; }
        public string Name { get; set; }              // "icon_1"
        public ulong FlashStart { get; set; }         // 數值位址
        public string FlashStartHex => $"0x{FlashStart:X}";

        // 供 UI 綁定顯示（直接用你現有的轉換結果）
        public ImageSource Thumbnail { get; set; }

        // 可選：如果你仍要保存原始 Bitmap 做其他計算
        public Bitmap RawBitmap { get; set; }
    }
}