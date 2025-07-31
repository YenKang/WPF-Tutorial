public partial class DrivePage : UserControl
{
    public DrivePage()
    {
        InitializeComponent();
        this.Loaded += DrivePage_Loaded;
    }

    private async void DrivePage_Loaded(object sender, RoutedEventArgs e)
    {
        // ✅ 取得主控 VM（非常重要）
        var mainVM = App.Current.MainWindow.DataContext as NovaCIDViewModel;
        if (mainVM == null) return;

        // ✅ 用主控 VM 建立 DrivePageViewModel
        var vm = new DrivePageViewModel(mainVM);
        this.DataContext = vm;

        // ✅ 等 UI 元件完成 Layout，再取得正確的按鈕座標
        await Dispatcher.InvokeAsync(() =>
        {
            var zoomInRect = GetButtonArea(ZoomInButton);
            var zoomOutRect = GetButtonArea(ZoomOutButton);
            vm.SetZoomButtons(zoomInRect, zoomOutRect);
        }, System.Windows.Threading.DispatcherPriority.Loaded);
    }

    private Rect GetButtonArea(FrameworkElement button)
    {
        var topLeft = button.TranslatePoint(new Point(0, 0), Application.Current.MainWindow);
        var bottomRight = button.TranslatePoint(
            new Point(button.ActualWidth, button.ActualHeight),
            Application.Current.MainWindow);

        return new Rect(topLeft, bottomRight);
    }
}



## Step1

```csharp
// ✅ ViewModel 內部只存範圍
private Rect _zoomInArea;
private Rect _zoomOutArea;

public void SetZoomButtons(Rect zoomInArea, Rect zoomOutArea)
{
    _zoomInArea = zoomInArea;
    _zoomOutArea = zoomOutArea;
}

// ✅ 判斷是否在 ZoomIn 區域
private bool IsTouchingZoomIn(int x, int y)
{
    return _zoomInArea.Contains(new Point(x, y));
}

// ✅ 判斷是否在 ZoomOut 區域
private bool IsTouchingZoomOut(int x, int y)
{
    return _zoomOutArea.Contains(new Point(x, y));
}
```


## Step2: View (code behind)

```csharp
private Rect GetButtonArea(FrameworkElement button)
{
    var topLeft = button.TranslatePoint(new Point(0, 0), Application.Current.MainWindow);
    var bottomRight = button.TranslatePoint(
        new Point(button.ActualWidth, button.ActualHeight),
        Application.Current.MainWindow);

    return new Rect(topLeft, bottomRight);
}
```

## Step3: DrivePage_Loaded 時設定按鈕範圍

```chsarp
private void DrivePage_Loaded(object sender, RoutedEventArgs e)
{
    var vm = new DrivePageViewModel(App.Current.MainVM);
    this.DataContext = vm;

    var zoomInRect = GetButtonArea(ZoomInButton);
    var zoomOutRect = GetButtonArea(ZoomOutButton);
    vm.SetZoomButtons(zoomInRect, zoomOutRect);
}
```

## Step4:

```csharp
public void OnFingerTouch(List<FingerStatus> fingers)
{
    foreach (var finger in fingers)
    {
        if (!finger.Driver) continue;

        var point = new Point(finger.X, finger.Y);

        if (IsTouchingZoomIn((int)point.X, (int)point.Y))
        {
            ZoomLevel++;
            UpdateMap();
        }
        else if (IsTouchingZoomOut((int)point.X, (int)point.Y))
        {
            ZoomLevel--;
            UpdateMap();
        }
    }
}
```