<StackPanel Margin="20" HorizontalAlignment="Right" VerticalAlignment="Top">
    <TextBlock x:Name="RightSliderTextX" Text="X: 2109" FontSize="40" HorizontalAlignment="Center" Margin="0,5,0,0"/>
    <Slider x:Name="RightKnob_SliderX" Minimum="0" Maximum="2560" Width="300" Value="2109"/>
    <TextBox x:Name="RightKnob_TextX" Width="200" FontSize="24" Margin="0,5,0,0" HorizontalAlignment="Center"/>

    <TextBlock x:Name="RightSliderTextY" Text="Y: 1117" FontSize="40" HorizontalAlignment="Center" Margin="0,20,0,0"/>
    <Slider x:Name="RightKnob_SliderY" Minimum="0" Maximum="1440" Width="300" Value="1117"/>
    <TextBox x:Name="RightKnob_TextY" Width="200" FontSize="24" Margin="0,5,0,0" HorizontalAlignment="Center"/>
</StackPanel>


private void UpdateRightKnobPosition(double x, double y)
{
    double outerRadius = 186;
    double innerRadius = 112;

    // 外圈
    Canvas.SetLeft(RightKnobOuter, x - outerRadius);
    Canvas.SetTop(RightKnobOuter, y - outerRadius);

    // 內圈
    Canvas.SetLeft(RightKnobInner, x - innerRadius);
    Canvas.SetTop(RightKnobInner, y - innerRadius);

    // 更新文字
    RightSliderTextX.Text = $"X: {Math.Round(x)}";
    RightSliderTextY.Text = $"Y: {Math.Round(y)}";
}

private void RightKnob_TextX_TextChanged(object sender, TextChangedEventArgs e)
{
    if (_isUpdatingRight) return;
    _isUpdatingRight = true;

    if (double.TryParse(RightKnob_TextX.Text, out double x))
    {
        x = Math.Max(0, Math.Min(2560, x));
        RightKnob_SliderX.Value = x;
        UpdateRightKnobPosition(x, RightKnob_SliderY.Value);
    }

    _isUpdatingRight = false;
}

private void RightKnob_TextY_TextChanged(object sender, TextChangedEventArgs e)
{
    if (_isUpdatingRight) return;
    _isUpdatingRight = true;

    if (double.TryParse(RightKnob_TextY.Text, out double y))
    {
        y = Math.Max(0, Math.Min(1440, y));
        RightKnob_SliderY.Value = y;
        UpdateRightKnobPosition(RightKnob_SliderX.Value, y);
    }

    _isUpdatingRight = false;
}