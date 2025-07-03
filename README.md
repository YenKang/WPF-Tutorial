private void RightKnob_SliderX_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (_isUpdatingRight) return;
    _isUpdatingRight = true;

    double x = e.NewValue;
    double y = RightKnob_SliderY.Value;

    RightKnob_TextX.Text = x.ToString("F0");
    UpdateRightKnobPosition(x, y);

    _isUpdatingRight = false;
}

private void RightKnob_SliderY_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (_isUpdatingRight) return;
    _isUpdatingRight = true;

    double x = RightKnob_SliderX.Value;
    double y = e.NewValue;

    RightKnob_TextY.Text = y.ToString("F0");
    UpdateRightKnobPosition(x, y);

    _isUpdatingRight = false;
}