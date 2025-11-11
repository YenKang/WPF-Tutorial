// View/ImagePickerWindow.xaml.cs
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using OSDIconFlashMap.Model;

public partial class ImagePickerWindow : Window
{
    private readonly List<ImageOption> _all;
    public ImageOption Selected { get; private set; }

    public ImagePickerWindow(IEnumerable<ImageOption> options, string preSelectedKey, ISet<string> usedKeys)
    {
        InitializeComponent();
        _all = options != null ? options.ToList() : new List<ImageOption>();

        foreach (var img in _all)
        {
            img.IsPreviouslySelected = (preSelectedKey != null && img.Key == preSelectedKey);
            img.IsUsedByOthers       = (usedKeys != null && usedKeys.Contains(img.Key)); // ★
            img.IsCurrentSelected    = false;
        }

        if (!string.IsNullOrEmpty(preSelectedKey))
        {
            var hit = _all.FirstOrDefault(x => x.Key == preSelectedKey);
            if (hit != null) { hit.IsCurrentSelected = true; Selected = hit; }
        }

        ic.ItemsSource = _all;
    }

    private void OnPick(object sender, MouseButtonEventArgs e)
    {
        var fe = sender as FrameworkElement;
        var op = fe?.DataContext as ImageOption;
        if (op == null) return;

        for (int i = 0; i < _all.Count; i++) _all[i].IsCurrentSelected = false;
        op.IsCurrentSelected = true;
        Selected = op;
        ic.Items.Refresh();
        DialogResult = true;
        Close();
    }
}