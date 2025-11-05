public void LoadCatalog(IEnumerable<IconModel> catalog)
{
    Icons.Clear();
    foreach (var icon in catalog)
    {
        // 規則：iconN → iconN_thumbnail
        if (string.IsNullOrEmpty(icon.ThumbnailKey))
            icon.ThumbnailKey = $"{icon.Name}_thumbnail";

        Icons.Add(icon);
    }

    Osds.Clear();
    for (int i = 1; i <= 40; i++)
    {
        var def = Icons.FirstOrDefault(x => x.Id == i) ?? Icons.FirstOrDefault();
        Osds.Add(new OsdModel { OsdName = $"OSD_{i}", SelectedIcon = def });
    }
}