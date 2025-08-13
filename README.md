private string BuildPosterPath(string fileName)
{
    var baseDir   = AppDomain.CurrentDomain.BaseDirectory;
    var posterDir = Path.Combine(baseDir, "debug", "posterModeImages");
    Directory.CreateDirectory(posterDir); // 沒有就建立
    return Path.Combine(posterDir, fileName);
}