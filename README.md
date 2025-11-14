public void LoadImagesFromFiles(IEnumerable<string> filePaths)
{
    var list = new List<string>();
    foreach (var file in filePaths)
    {
        if (File.Exists(file))
            list.Add(file);
    }

    if (list.Count == 0)
        return;

    list.Sort(StringComparer.OrdinalIgnoreCase);

    var opts = new List<ImageOption>();
    int id = 1;

    foreach (var path in list)
    {
        var name = Path.GetFileNameWithoutExtension(path);
        opts.Add(new ImageOption
        {
            Id = id++,
            Name = name,
            Key = name,
            ThumbnailKey = name + "_thumbnail",
            ImagePath = path
        });
    }

    LoadImages(opts.ToArray());
}