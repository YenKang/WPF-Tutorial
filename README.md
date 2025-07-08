private void SaveLeftKnobPosition_Click(object sender, RoutedEventArgs e)
{
    // 取得 Left Knob 當前座標
    double x = Canvas.GetLeft(LeftKnobEllipse) + LeftKnobEllipse.Width / 2;
    double y = Canvas.GetTop(LeftKnobEllipse) + LeftKnobEllipse.Height / 2;

    // 存到 ViewModel
    _viewModel.LeftKnobX = x;
    _viewModel.LeftKnobY = y;

    System.Diagnostics.Debug.WriteLine($"[儲存] LeftKnobX: {x}, LeftKnobY: {y}");

    // 隱藏 StackPanel（Canvas 繼續顯示）
    _viewModel.IsAdjustPanelVisible = false;
}

<Button Content="存取 Left Knob 位置"
        Width="150"
        Height="50"
        HorizontalAlignment="Center"
        VerticalAlignment="Bottom"
        Margin="0,0,0,20"
        Click="SaveLeftKnobPosition_Click"/>