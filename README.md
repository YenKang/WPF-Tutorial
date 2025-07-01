<Grid>
    <!-- 原本 Canvas -->
    <Canvas x:Name="MainCanvas" Width="2560" Height="1440" Background="Transparent" IsHitTestVisible="False">
        <Ellipse x:Name="LeftKnobOuter"
                 Width="371" Height="371"
                 Fill="LightGray"
                 Stroke="Red"
                 StrokeThickness="4" />
        <Ellipse x:Name="LeftKnobInner"
                 Width="223" Height="223"
                 Fill="Black"
                 Stroke="Yellow"
                 StrokeThickness="2" />
    </Canvas>

    <!-- ⭐ 滑桿與座標 StackPanel -->
    <StackPanel Orientation="Horizontal" VerticalAlignment="Top" HorizontalAlignment="Center" Margin="20">
        <StackPanel>
            <TextBlock Text="X" HorizontalAlignment="Center"/>
            <Slider x:Name="SliderX" Minimum="0" Maximum="2560" Width="300" Value="445"/>
            <TextBlock x:Name="TextX" Text="X: 445" HorizontalAlignment="Center" Margin="0,5,0,0"/>
        </StackPanel>
        <StackPanel Margin="20,0,0,0">
            <TextBlock Text="Y" HorizontalAlignment="Center"/>
            <Slider x:Name="SliderY" Minimum="0" Maximum="1440" Width="300" Value="1113"/>
            <TextBlock x:Name="TextY" Text="Y: 1113" HorizontalAlignment="Center" Margin="0,5,0,0"/>
        </StackPanel>
    </StackPanel>
</Grid>


```csharp
private void PositionLeftKnob()
{
    double centerX = SliderX.Value;
    double centerY = SliderY.Value;

    double outerRadius = 186;
    double innerRadius = 112;

    Canvas.SetLeft(LeftKnobOuter, centerX - outerRadius);
    Canvas.SetTop(LeftKnobOuter, centerY - outerRadius);

    Canvas.SetLeft(LeftKnobInner, centerX - innerRadius);
    Canvas.SetTop(LeftKnobInner, centerY - innerRadius);

    // ⭐ 更新 TextBlock 顯示
    TextX.Text = $"X: {Math.Round(centerX)}";
    TextY.Text = $"Y: {Math.Round(centerY)}";
}

public NovaCIDView()
{
    InitializeComponent();

    PositionLeftKnob();

    SliderX.ValueChanged += (s, e) => PositionLeftKnob();
    SliderY.ValueChanged += (s, e) => PositionLeftKnob();
}
```