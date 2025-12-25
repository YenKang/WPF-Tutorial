using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Runtime.InteropServices;

public static class ImageExporter
{
    // 你的需求：H/V 各自 range + step，輸出 (W,H) 笛卡兒積
    public static void ExportNoiseBmpBySizeGrid(
        int hStart, int hEnd, int hStep,
        int vStart, int vEnd, int vStep,
        string outputDir,
        bool deterministicBySize = true
    )
    {
        if (hStep <= 0) throw new ArgumentOutOfRangeException(nameof(hStep));
        if (vStep <= 0) throw new ArgumentOutOfRangeException(nameof(vStep));
        if (hEnd < hStart) throw new ArgumentException("hEnd must be >= hStart");
        if (vEnd < vStart) throw new ArgumentException("vEnd must be >= vStart");

        Directory.CreateDirectory(outputDir);

        // deterministicBySize=false：每次跑都完全不同
        var globalRng = deterministicBySize ? null : new Random();

        for (int w = hStart; w <= hEnd; w += hStep)
        {
            for (int h = vStart; h <= vEnd; h += vStep)
            {
                var rng = deterministicBySize ? new Random(HashSizeToSeed(w, h)) : globalRng;

                using (var bmp = GenerateRandomPixelBitmapFast32bpp(w, h, rng))
                {
                    string fileName = $"noise_{w}x{h}.bmp";
                    string path = Path.Combine(outputDir, fileName);
                    bmp.Save(path, ImageFormat.Bmp); // ✅ BMP 最快
                }
            }
        }
    }

    // ✅ 超快版本：32bpp + LockBits + NextBytes 一次塞滿
    private static Bitmap GenerateRandomPixelBitmapFast32bpp(int width, int height, Random rng)
    {
        var bmp = new Bitmap(width, height, PixelFormat.Format32bppArgb);
        var rect = new Rectangle(0, 0, width, height);

        BitmapData data = null;
        try
        {
            data = bmp.LockBits(rect, ImageLockMode.WriteOnly, PixelFormat.Format32bppArgb);

            int stride = data.Stride;
            int totalBytes = stride * height;
            byte[] buffer = new byte[totalBytes];

            // 一次填滿：內容是隨機 byte → 就是你要的「彩色雜訊」
            rng.NextBytes(buffer);

            // 但 Alpha 也會被亂填，可能造成透明/怪色；我們把 A 固定成 255
            // ARGB 在記憶體通常是 BGRA 排列（Format32bppArgb）
            for (int i = 3; i < totalBytes; i += 4)
                buffer[i] = 255;

            Marshal.Copy(buffer, 0, data.Scan0, totalBytes);
            return bmp;
        }
        finally
        {
            if (data != null) bmp.UnlockBits(data);
        }
    }

    private static int HashSizeToSeed(int w, int h)
    {
        unchecked
        {
            int seed = 17;
            seed = seed * 31 + w;
            seed = seed * 31 + h;
            return seed;
        }
    }
}
