

```
public static void ExportNoiseSeriesBySizeGrid(
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

    // deterministicBySize=false：每次都不同
    var globalRng = deterministicBySize ? null : new Random();

    for (int w = hStart; w <= hEnd; w += hStep)
    {
        for (int h = vStart; h <= vEnd; h += vStep)
        {
            // deterministicBySize=true：同一個 (w,h) 永遠同一張（好debug）
            var rng = deterministicBySize ? new Random(HashSizeToSeed(w, h)) : globalRng;

            using (var bmp = GenerateRandomPixelBitmap(w, h, rng))
            {
                string fileName = $"noise_{w}x{h}.png";
                string path = Path.Combine(outputDir, fileName);
                bmp.Save(path, ImageFormat.Png);
            }
        }
    }
}

private static int HashSizeToSeed(int w, int h)
{
    unchecked
    {
        // 簡單穩定的 seed 組合，避免 (w,h) 太小時 seed 重複
        int seed = 17;
        seed = seed * 31 + w;
        seed = seed * 31 + h;
        return seed;
    }
}
```


```
ImageExporter.ExportNoiseSeriesBySizeGrid(
    hStart: 10, hEnd: 12, hStep: 1,
    vStart: 7,  vEnd: 8,  vStep: 1,
    outputDir: @"D:\Temp\NoiseImages",
    deterministicBySize: true
);
```
