
```xaml
<StackPanel Margin="20">
    <!-- X Slider -->
    <Slider x:Name="LeftKnob_SliderX" Minimum="0" Maximum="2560" Width="300" Value="445" ValueChanged="LeftKnob_SliderX_ValueChanged"/>
    <TextBlock x:Name="LeftSliderTextX" Text="X: 445" FontSize="32" HorizontalAlignment="Center" Margin="0,5,0,0"/>

    <!-- X TextBox -->
    <TextBox x:Name="LeftKnob_TextX" Width="120" FontSize="28" HorizontalAlignment="Center" Margin="0,5,0,0"
             TextChanged="LeftKnob_TextX_TextChanged"/>

    <!-- Y Slider -->
    <Slider x:Name="LeftKnob_SliderY" Minimum="0" Maximum="1440" Width="300" Value="1113" ValueChanged="LeftKnob_SliderY_ValueChanged"/>
    <TextBlock x:Name="LeftSliderTextY" Text="Y: 1113" FontSize="32" HorizontalAlignment="Center" Margin="0,5,0,0"/>

    <!-- Y TextBox -->
    <TextBox x:Name="LeftKnob_TextY" Width="120" FontSize="28" HorizontalAlignment="Center" Margin="0,5,0,0"
             TextChanged="LeftKnob_TextY_TextChanged"/>
</StackPanel>
```

```csharp
private void LeftKnob_SliderX_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (LeftKnob_TextX != null)
        LeftKnob_TextX.Text = Math.Round(e.NewValue).ToString();

    UpdateLeftKnobPosition(e.NewValue, LeftKnob_SliderY.Value);
}

private void LeftKnob_SliderY_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (LeftKnob_TextY != null)
        LeftKnob_TextY.Text = Math.Round(e.NewValue).ToString();

    UpdateLeftKnobPosition(LeftKnob_SliderX.Value, e.NewValue);
}

private void LeftKnob_TextX_TextChanged(object sender, TextChangedEventArgs e)
{
    if (LeftKnob_TextX == null || LeftKnob_SliderX == null) return;

    if (double.TryParse(LeftKnob_TextX.Text, out double value))
    {
        value = Math.Max(0, Math.Min(2560, value)); // 限制範圍
        LeftKnob_SliderX.Value = value;
        UpdateLeftKnobPosition(value, LeftKnob_SliderY.Value);
    }
}

private void LeftKnob_TextY_TextChanged(object sender, TextChangedEventArgs e)
{
    if (LeftKnob_TextY == null || LeftKnob_SliderY == null) return;

    if (double.TryParse(LeftKnob_TextY.Text, out double value))
    {
        value = Math.Max(0, Math.Min(1440, value)); // 限制範圍
        LeftKnob_SliderY.Value = value;
        UpdateLeftKnobPosition(LeftKnob_SliderX.Value, value);
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

    // 若有中心點（紅點），也可以更新
    Canvas.SetLeft(LeftKnobCenterPoint, x - 10);
    Canvas.SetTop(LeftKnobCenterPoint, y - 10);

    // 同步更新 TextBlock 顯示
    LeftSliderTextX.Text = $"X: {Math.Round(x)}";
    LeftSliderTextY.Text = $"Y: {Math.Round(y)}";
}
```

```csharp
this.Loaded += (s, e) =>
{
    LeftKnob_TextX.Text = Math.Round(LeftKnob_SliderX.Value).ToString();
    LeftKnob_TextY.Text = Math.Round(LeftKnob_SliderY.Value).ToString();

    UpdateLeftKnobPosition(LeftKnob_SliderX.Value, LeftKnob_SliderY.Value);
};
```