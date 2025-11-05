// 放在檔案頂端（或現有 using 區塊）
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;

using OSDIconFlashMap.Model;  // IconModel
using OSDIconFlashMap.View;   // IconFlashMapWindow

// 事件處理：開啟 Icon Flash Map（縮圖先略過）
private void OpenIconFlashMapTool_Click(object sender, RoutedEventArgs e)
{
    // 1) 找到畫面上的 OSDButtonList 根（優先用你有命名的元素，否則找型別名）
    var legacyRoot =
        (this as DependencyObject) is FrameworkElement feByName && feByName.FindName("OsdButtonListRoot") is FrameworkElement namedRoot
            ? namedRoot
            : FindVisualDescendantByTypeName(this as DependencyObject, "OSDButtonList") as FrameworkElement;

    if (legacyRoot == null)
    {
        MessageBox.Show("找不到 OSDButtonList（或 OsdButtonListRoot）。");
        return;
    }

    // 2) 從每一列 OSDButton 讀取 flash start address
    var icons = new List<IconModel>();
    int idx = 0;

    foreach (var row in FindVisualChildren<FrameworkElement>(legacyRoot)
                        .Where(x => x.GetType().Name == "OSDButton"))
    {
        // Address 來源：優先 bmp1_addr，否則 bmpl_addr
        var addrBox = row.FindName("bmp1_addr") as TextBox ?? row.FindName("bmpl_addr") as TextBox;
        ulong addr = ParseHex(addrBox?.Text);

        var name = $"icon_{++idx}";
        icons.Add(new IconModel
        {
            Id = idx,
            Name = name,
            FlashStart = addr,
            // 縮圖先不載入，只放對應鍵名（未來載 Bitmap 時依規則 name + "_thumbnail"）
            ThumbnailKey = $"{name}_thumbnail"
        });
    }

    // 3) 開視窗並傳入 catalog
    var win = new IconFlashMapWindow();
    win.LoadCatalog(icons);

    // 這個 click 事件多半不在 Window 內，用這招安全取得 Owner
    var owner = Window.GetWindow(this as DependencyObject);
    if (owner != null) win.Owner = owner;

    win.Show();
}

/* ===================== 小工具區 ===================== */

private static ulong ParseHex(string s)
{
    if (string.IsNullOrWhiteSpace(s)) return 0;
    s = (s.StartsWith("0x", StringComparison.OrdinalIgnoreCase)) ? s.Substring(2) : s;
    return ulong.TryParse(s, System.Globalization.NumberStyles.HexNumber, null, out var v) ? v : 0;
}

private static IEnumerable<T> FindVisualChildren<T>(DependencyObject d) where T : DependencyObject
{
    if (d == null) yield break;
    for (int i = 0; i < VisualTreeHelper.GetChildrenCount(d); i++)
    {
        var child = VisualTreeHelper.GetChild(d, i);
        if (child is T t) yield return t;
        foreach (var c in FindVisualChildren<T>(child)) yield return c;
    }
}

private static DependencyObject FindVisualDescendantByTypeName(DependencyObject root, string typeName)
{
    if (root == null) return null;
    for (int i = 0; i < VisualTreeHelper.GetChildrenCount(root); i++)
    {
        var ch = VisualTreeHelper.GetChild(root, i);
        if (ch.GetType().Name == typeName) return ch;
        var deep = FindVisualDescendantByTypeName(ch, typeName);
        if (deep != null) return deep;
    }
    return null;
}

