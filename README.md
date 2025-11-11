// Model/ImageOption.cs
public class ImageOption
{
    public string Key { get; set; }        // "image_1"
    public string Name { get; set; }
    public string ImagePath { get; set; }

    public bool IsPreviouslySelected { get; set; } // 本列上次選
    public bool IsCurrentSelected { get; set; }    // 本次點選
    public bool IsUsedByOthers { get; set; }       // 其他列已使用 ← 新增
}