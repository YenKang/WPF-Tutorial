public ImagePickerWindow(IEnumerable<ImageOption> options, string preSelectedKey, ISet<string> usedKeys)
{
    InitializeComponent();

    _all = options?.ToList() ?? new List<ImageOption>();
    foreach (var img in _all)
    {
        img.IsUsedByOthers     = usedKeys != null && usedKeys.Contains(img.Key);
        img.IsPreviouslySelected = preSelectedKey != null && img.Key == preSelectedKey;
        img.IsCurrentSelected  = false;
    }
    if (!string.IsNullOrEmpty(preSelectedKey))
    {
        var hit = _all.FirstOrDefault(x => x.Key == preSelectedKey);
        if (hit != null) { hit.IsCurrentSelected = true; Selected = hit; }
    }

    // ✅ 關鍵三行：先拔、清、再綁
    ic.ItemsSource = null;
    ic.Items.Clear();
    ic.ItemsSource = _all;
}