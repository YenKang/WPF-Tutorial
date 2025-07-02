<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="30,0">
    <!-- 其他 App icon 按鈕 -->
    
    <Button Content="Test RawCounter 100"
            Click="TestRaw100_Click"
            Width="250"
            Height="60"
            Margin="20"/>
</StackPanel>

private void TestRaw100_Click(object sender, RoutedEventArgs e)
{
    UpdateRightKnobArc(100);
}

public void UpdateRightKnobArc(int rawCounter)
{
    double angle = (rawCounter / 256.0) * 360.0;

    double centerX = 2115;
    double centerY = 1113;
    double radius = 186;

    var geom = new StreamGeometry();
    using (var ctx = geom.Open())
    {
        double startAngle = -90;
        double endAngle = startAngle + angle;

        double startX = centerX + radius * Math.Cos(startAngle * Math.PI / 180);
        double startY = centerY + radius * Math.Sin(startAngle * Math.PI / 180);

        double endX = centerX + radius * Math.Cos(endAngle * Math.PI / 180);
        double endY = centerY + radius * Math.Sin(endAngle * Math.PI / 180);

        bool isLargeArc = angle > 180;

        ctx.BeginFigure(new Point(startX, startY), false, false);
        ctx.ArcTo(new Point(endX, endY),
                  new Size(radius, radius),
                  0,
                  isLargeArc,
                  SweepDirection.Clockwise,
                  true,
                  false);
    }

    geom.Freeze();
    RightKnobArc.Data = geom;
}


<Path x:Name="RightKnobArc"
      Stroke="Cyan"
      StrokeThickness="12"
      Visibility="Visible"/>