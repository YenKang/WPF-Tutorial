var baseDir = AppDomain.CurrentDomain.BaseDirectory;

// 1) 淨化檔名（避免 JSON 裡有多餘空白或 ./）
var fileName = (node.Icon ?? "").Trim();
fileName = System.IO.Path.GetFileName(fileName);

// 2) 組成完整路徑
var absPath = System.IO.Path.Combine(baseDir, "Assets", "Patterns", _profile.IcName, fileName);

// 3) 若檔案不存在，用 placeholder
Uri iconUri;
if (System.IO.File.Exists(absPath))
{
    iconUri = new Uri(absPath, UriKind.Absolute);
}
else
{
    var placeholder = System.IO.Path.Combine(baseDir, "Assets", "placeholder.png");
    iconUri = new Uri(placeholder, UriKind.Absolute);
    System.Diagnostics.Debug.WriteLine($"[Missing] {absPath}");
}