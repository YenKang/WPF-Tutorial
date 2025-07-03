
```xaml
<Window x:Class="YourNamespace.NovaCIDView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="NovaCIDView" Width="2560" Height="1440"
        Background="Black">

    <Grid>
        <!-- 左旋鈕控制 UI -->
        <StackPanel HorizontalAlignment="Left" VerticalAlignment="Top" Margin="20">
            <!-- X Slider -->
            <Slider x:Name="LeftKnob_SliderX" Minimum="0" Maximum="2560" Width="300" Value="445"/>
            <TextBox x:Name="LeftKnob_TextX" Text="445" Width="120" FontSize="28" Margin="0,5,0,0"/>

            <!-- Y Slider -->
            <Slider x:Name="LeftKnob_SliderY" Minimum="0" Maximum="1440" Width="300" Value="1113"/>
            <TextBox x:Name="LeftKnob_TextY" Text="1113" Width="120" FontSize="28" Margin="0,5,0,0"/>
        </StackPanel>

        <!-- Knob canvas -->
        <Canvas x:Name="KnobCanvas" Width="2560" Height="1440" Background="Transparent">
            <Ellipse x:Name="LeftKnobOuter" Width="371" Height="371" Fill="LightGray" Stroke="Red" StrokeThickness="4"/>
            <Ellipse x:Name="LeftKnobInner" Width="223" Height="223" Fill="Black" Stroke="Yellow" StrokeThickness="2"/>
            <Ellipse x:Name="LeftKnobCenterPoint" Width="20" Height="20" Fill="Red"/>
        </Canvas>
    </Grid>
</Window>
```

```csharp
using System;
using System.Windows;
using System.Windows.Controls;

namespace YourNamespace
{
    public partial class NovaCIDView : Window
    {
        public NovaCIDView()
        {
            InitializeComponent();

            // 🔥 只在 Loaded 再去註冊事件，確保所有元件都已初始化
            this.Loaded += NovaCIDView_Loaded;
        }

        private void NovaCIDView_Loaded(object sender, RoutedEventArgs e)
        {
            // Slider → TextBox
            LeftKnob_SliderX.ValueChanged += LeftKnob_SliderX_ValueChanged;
            LeftKnob_SliderY.ValueChanged += LeftKnob_SliderY_ValueChanged;

            // TextBox → Slider
            LeftKnob_TextX.TextChanged += LeftKnob_TextX_TextChanged;
            LeftKnob_TextY.TextChanged += LeftKnob_TextY_TextChanged;

            // 初始同步
            LeftKnob_TextX.Text = Math.Round(LeftKnob_SliderX.Value).ToString();
            LeftKnob_TextY.Text = Math.Round(LeftKnob_SliderY.Value).ToString();
            UpdateLeftKnobPosition(LeftKnob_SliderX.Value, LeftKnob_SliderY.Value);
        }

        private void LeftKnob_SliderX_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            if (LeftKnob_TextX == null) return;

            LeftKnob_TextX.Text = Math.Round(e.NewValue).ToString();
            UpdateLeftKnobPosition(e.NewValue, LeftKnob_SliderY?.Value ?? 0);
        }

        private void LeftKnob_SliderY_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            if (LeftKnob_TextY == null) return;

            LeftKnob_TextY.Text = Math.Round(e.NewValue).ToString();
            UpdateLeftKnobPosition(LeftKnob_SliderX?.Value ?? 0, e.NewValue);
        }

        private void LeftKnob_TextX_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (LeftKnob_SliderX == null) return;

            if (double.TryParse(LeftKnob_TextX.Text, out double value))
            {
                value = Math.Max(0, Math.Min(2560, value));
                LeftKnob_SliderX.Value = value;
                UpdateLeftKnobPosition(value, LeftKnob_SliderY?.Value ?? 0);
            }
        }

        private void LeftKnob_TextY_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (LeftKnob_SliderY == null) return;

            if (double.TryParse(LeftKnob_TextY.Text, out double value))
            {
                value = Math.Max(0, Math.Min(1440, value));
                LeftKnob_SliderY.Value = value;
                UpdateLeftKnobPosition(LeftKnob_SliderX?.Value ?? 0, value);
            }
        }

        private void UpdateLeftKnobPosition(double x, double y)
        {
            double outerRadius = 186;
            double innerRadius = 112;
            double centerRadius = 10;

            Canvas.SetLeft(LeftKnobOuter, x - outerRadius);
            Canvas.SetTop(LeftKnobOuter, y - outerRadius);

            Canvas.SetLeft(LeftKnobInner, x - innerRadius);
            Canvas.SetTop(LeftKnobInner, y - innerRadius);

            Canvas.SetLeft(LeftKnobCenterPoint, x - centerRadius);
            Canvas.SetTop(LeftKnobCenterPoint, y - centerRadius);
        }
    }
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