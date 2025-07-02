```csharp
private void UpdateRightKnobArc(int rawCounter)
{
    double angle = (rawCounter / 256.0) * 360.0;

    double centerX = 2115; // ✅ 你之前算好的 Right Knob 圓心 X
    double centerY = 1113; // ✅ 你之前算好的 Right Knob 圓心 Y
    double radius = 186;

    var geom = new StreamGeometry();
    using (var ctx = geom.Open())
    {
        double startAngle = -90;  // 從上面 12 點鐘方向開始
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
```