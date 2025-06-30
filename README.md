private void AddLeftKnobGlow()
{
    // 定義半徑
    double radius = 150; // 你可以自行調整
    double centerX = 445;
    double centerY = 111;

    // 建立 Ellipse
    Ellipse leftGlow = new Ellipse
    {
        Width = radius * 2,
        Height = radius * 2,
        Fill = new SolidColorBrush(Color.FromArgb(80, 0, 120, 255)), // 半透明藍光
        Stroke = Brushes.Blue,
        StrokeThickness = 2
    };

    // 設定在 Canvas 上的絕對位置
    Canvas.SetLeft(leftGlow, centerX - radius);
    Canvas.SetTop(leftGlow, centerY - radius);

    // 加進 Canvas
    MainCanvas.Children.Add(leftGlow);
}