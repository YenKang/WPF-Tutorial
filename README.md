using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;

public static class ImageExporter
{
    /// <summary>
    /// 批次輸出：v = start..end (step)，每張都是隨機像素圖，存到 outputDir
    /// </summary>
    public static void ExportNoiseSeries(
        int start,
        int end,
        int step,
        int width,
        int height,
        string outputDir,
        bool deterministicByValue = true
    )
    {
        if (step <= 0) throw new ArgumentOutOfRangeException(nameof(step));
        if (width <= 0 || height <= 0) throw new ArgumentOutOfRangeException("width/height must be > 0");
        if (end < start) throw new ArgumentException("end must be >= start");

        Directory.CreateDirectory(outputDir);

        // deterministicByValue=false：每次跑都不一樣
        var globalRng = deterministicByValue ? null : new Random();

        for (int v = start; v <= end; v += step)
        {
            // deterministicByValue=true：同一個 v 永遠產同一張（好 debug）
            var rng = deterministicByValue ? new Random(v) : globalRng;

            using (var bmp = GenerateRandomPixelBitmap(width, height, rng))
            {
                string fileName = $"noise_{v:D3}_{width}x{height}.png";
                string path = Path.Combine(outputDir, fileName);
                bmp.Save(path, ImageFormat.Png);
            }
        }
    }

    /// <summary>
    /// 單張圖：每個像素都是隨機 RGB
    /// </summary>
    private static Bitmap GenerateRandomPixelBitmap(int width, int height, Random rng)
    {
        var bmp = new Bitmap(width, height, PixelFormat.Format24bppRgb);

        for (int y = 0; y < height; y++)
        {
            for (int x = 0; x < width; x++)
            {
                var c = Color.FromArgb(
                    rng.Next(256), // R
                    rng.Next(256), // G
                    rng.Next(256)  // B
                );
                bmp.SetPixel(x, y, c);
            }
        }

        return bmp;
    }
}
