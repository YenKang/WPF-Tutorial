public partial class ImagePickerWindow : Window
{
    private List<ImageOption> _all = new List<ImageOption>();
    public ImageOption Selected { get; private set; }

    // ✅ 新建構子：多帶一個 preSelectedKey
    public ImagePickerWindow(IEnumerable<ImageOption> options, string preSelectedKey)
    {
        InitializeComponent();

        _all = options != null ? new List<ImageOption>(options) : new List<ImageOption>();

        // 標記「上次選過」
        for (int i = 0; i < _all.Count; i++)
            _all[i].IsPreviouslySelected = (_all[i].Key == preSelectedKey);

        lb.ItemsSource = _all;

        // 預選中（高亮）
        if (!string.IsNullOrEmpty(preSelectedKey))
        {
            var hit = _all.FirstOrDefault(x => x.Key == preSelectedKey);
            if (hit != null) lb.SelectedItem = hit;
        }

        lb.SelectionChanged += Lb_SelectionChanged;
    }

    private void Lb_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        var op = lb.SelectedItem as ImageOption;
        if (op != null) { Selected = op; txtStatus.Text = "選擇：" + op.Name; }
        else { Selected = null; txtStatus.Text = "未選擇"; }
    }

    private void lb_MouseDoubleClick(object sender, MouseButtonEventArgs e) { ConfirmIfAny(); }
    private void Window_KeyDown(object sender, KeyEventArgs e)
    {
        if (e.Key == Key.Enter) { ConfirmIfAny(); e.Handled = true; }
        else if (e.Key == Key.Escape) { Cancel_Click(this, new RoutedEventArgs()); e.Handled = true; }
    }
    private void Ok_Click(object sender, RoutedEventArgs e) { ConfirmIfAny(); }
    private void ConfirmIfAny()
    {
        var op = lb.SelectedItem as ImageOption;
        if (op != null) { Selected = op; DialogResult = true; Close(); }
    }
    private void Cancel_Click(object sender, RoutedEventArgs e) { Selected = null; DialogResult = false; Close(); }
    protected override void OnClosing(CancelEventArgs e)
    {
        if (DialogResult == null) DialogResult = false;
        base.OnClosing(e);
    }
}