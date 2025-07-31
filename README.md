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