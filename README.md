```csharp
private void SaveKnobPosition_Click(object sender, RoutedEventArgs e)
{
    // 取得 Canvas 上 Knob 當前座標
    double x = Canvas.GetLeft(KnobEllipse) + KnobEllipse.Width / 2;
    double y = Canvas.GetTop(KnobEllipse) + KnobEllipse.Height / 2;

    // 存到 ViewModel
    _viewModel.KnobX = x;
    _viewModel.KnobY = y;

    System.Diagnostics.Debug.WriteLine($"[儲存] KnobX: {x}, KnobY: {y}");

    // 只隱藏 StackPanel
    _viewModel.IsAdjustPanelVisible = false;
}
```