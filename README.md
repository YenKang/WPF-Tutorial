<StackPanel Orientation="Vertical" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="30">
    <!-- X 軸 -->
    <TextBlock Text="X:" FontSize="24" Foreground="White"/>
    <Slider x:Name="LeftKnob_SliderX" Minimum="0" Maximum="2560" Width="300" Value="445" ValueChanged="LeftKnob_Slider_ValueChanged"/>
    <TextBox x:Name="LeftKnob_TextX" Width="100" Text="445" FontSize="24" Margin="0,5,0,10" TextChanged="LeftKnob_TextX_TextChanged"/>

    <!-- Y 軸 -->
    <TextBlock Text="Y:" FontSize="24" Foreground="White"/>
    <Slider x:Name="LeftKnob_SliderY" Minimum="0" Maximum="1440" Width="300" Value="1113" ValueChanged="LeftKnob_Slider_ValueChanged"/>
    <TextBox x:Name="LeftKnob_TextY" Width="100" Text="1113" FontSize="24" Margin="0,5,0,0" TextChanged="LeftKnob_TextY_TextChanged"/>
</StackPanel>

private void LeftKnob_Slider_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (LeftKnob_TextX != null && LeftKnob_TextY != null)
    {
        double x = LeftKnob_SliderX.Value;
        double y = LeftKnob_SliderY.Value;

        LeftKnob_TextX.Text = Math.Round(x).ToString();
        LeftKnob_TextY.Text = Math.Round(y).ToString();

        UpdateLeftKnobPosition(x, y);
    }
}


private void LeftKnob_TextX_TextChanged(object sender, TextChangedEventArgs e)
{
    if (double.TryParse(LeftKnob_TextX.Text, out double value))
    {
        // 防呆：限制在範圍內
        value = Math.Max(0, Math.Min(2560, value));
        LeftKnob_SliderX.Value = value;
    }
}

private void UpdateLeftKnobPosition(double x, double y)
{
    double outerRadius = 186;
    double innerRadius = 112;

    Canvas.SetLeft(LeftKnobOuter, x - outerRadius);
    Canvas.SetTop(LeftKnobOuter, y - outerRadius);

    Canvas.SetLeft(LeftKnobInner, x - innerRadius);
    Canvas.SetTop(LeftKnobInner, y - innerRadius);
}