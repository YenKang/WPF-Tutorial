private void UpdateLeftKnobPosition(double centerX, double centerY)
{
    double outerRadius = 186;
    double innerRadius = 112;

    Canvas.SetLeft(LeftKnobOuter, centerX - outerRadius);
    Canvas.SetTop(LeftKnobOuter, centerY - outerRadius);

    Canvas.SetLeft(LeftKnobInner, centerX - innerRadius);
    Canvas.SetTop(LeftKnobInner, centerY - innerRadius);

    Canvas.SetLeft(LeftKnobCenterPoint, centerX - 10);
    Canvas.SetTop(LeftKnobCenterPoint, centerY - 10);

    LeftSliderTextX.Text = $"X: {Math.Round(centerX)}";
    LeftSliderTextY.Text = $"Y: {Math.Round(centerY)}";
}

