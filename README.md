public partial class ImagePickerWindow : Window
{
    private readonly List<ImageOption> _all;
    public ImageOption Selected { get; private set; }

    public ImagePickerWindow(IEnumerable<ImageOption> options, string preSelectedKey, ISet<string> usedKeys)
    {
        InitializeComponent();

        _all = options != null ? options.ToList() : new List<ImageOption>();

        // 標記「本列上次選過」與「其他列已使用」
        foreach (var img in _all)
        {
            img.IsPreviouslySelected = (preSelectedKey != null && img.Key == preSelectedKey);
            img.IsUsedByOthers = (usedKeys != null && usedKeys.Contains(img.Key));
            img.IsCurrentSelected = false;
        }

        // 預選中（讓它一進來就藍框）
        if (!string.IsNullOrEmpty(preSelectedKey))
        {
            var hit = _all.FirstOrDefault(x => x.Key == preSelectedKey);
            if (hit != null)
            {
                hit.IsCurrentSelected = true;
                Selected = hit;
            }
        }

        ic.ItemsSource = _all;
        PreviewKeyDown += ImagePickerWindow_PreviewKeyDown;
    }

    // 其餘 OnPick / Enter / Esc 寫法維持你現有版本
}