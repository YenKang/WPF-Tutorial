private void PositionRightKnob()
{
    double centerX = SliderX_Right.Value;
    double centerY = SliderY_Right.Value;

    double outerRadius = 186;
    double innerRadius = 112;

    Canvas.SetLeft(RightKnobOuter, centerX - outerRadius);
    Canvas.SetTop(RightKnobOuter, centerY - outerRadius);

    Canvas.SetLeft(RightKnobInner, centerX - innerRadius);
    Canvas.SetTop(RightKnobInner, centerY - innerRadius);

    TextX_Right.Text = $"X: {Math.Round(centerX)}";
    TextY_Right.Text = $"Y: {Math.Round(centerY)}";
}