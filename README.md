private void PositionLeftKnob()
{
    double centerX = 445;
    double centerY = 1113; // ✅ 根據 15 cm 測量

    double outerRadius = 186;
    double innerRadius = 112;

    Canvas.SetLeft(LeftKnobOuter, centerX - outerRadius);
    Canvas.SetTop(LeftKnobOuter, centerY - outerRadius);

    Canvas.SetLeft(LeftKnobInner, centerX - innerRadius);
    Canvas.SetTop(LeftKnobInner, centerY - innerRadius);
}