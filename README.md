```xaml
<Grid Width="2560" Height="1440" Background="#0F1A24">
    <Grid.RowDefinitions>
        <RowDefinition Height="3*" />
        <RowDefinition Height="2*" />
    </Grid.RowDefinitions>

    <!-- 畫面主體（Page 切換） -->
    <ContentControl Grid.Row="0" Content="{Binding CurrentPage}" />

    <!-- 下方 Knob + Dock 區 -->
    <Grid Grid.Row="1">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="9*" />
            <ColumnDefinition Width="16*" />
            <ColumnDefinition Width="9*" />
        </Grid.ColumnDefinitions>

        <!-- 主駕 Knob 區 -->
        <Grid Grid.Column="0" Margin="15" RenderTransformOrigin="0.5,0.5">
            <Grid.RenderTransform>
                <RotateTransform x:Name="LeftKnobRotation" Angle="0" />
            </Grid.RenderTransform>
            <!-- 外圈銀色圓 -->
            <Ellipse Width="300" Height="300" Fill="LightGray" Stroke="Gray" StrokeThickness="4" />
            <!-- 內圈黑色圓 -->
            <Ellipse Width="180" Height="180" Fill="Black" />
        </Grid>

        <!-- Dock App 區 -->
        <StackPanel Grid.Column="1" Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="30,0">
            <Button Content="🏠" Width="180" Height="180" Margin="50" FontSize="120" Command="{Binding NavigateToHomeCommand}" />
        </StackPanel>

        <!-- 副駕 Knob 區（可擴充） -->
    </Grid>

    <!-- Glow（放同層 Grid） -->
    <Ellipse x:Name="LeftKnobGlow"
             Width="300" Height="300"
             Fill="#500078FF"
             Stroke="Blue"
             StrokeThickness="2"
             Visibility="Collapsed"/>
</Grid>
```


```csharp
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Shapes;

namespace NovaCID
{
    public partial class NovaCIDView : UserControl
    {
        public NovaCIDView()
        {
            InitializeComponent();

            // 固定解析度（模擬 15.6 吋）
            var win = Window.GetWindow(this);
            if (win != null)
            {
                win.Width = 1472;
                win.Height = 829;
                win.WindowStartupLocation = WindowStartupLocation.CenterScreen;
            }

            ShowLeftKnobGlow();
        }

        private void ShowLeftKnobGlow()
        {
            double radius = 150;
            double centerX = 445;
            double centerY = 111;

            Canvas.SetLeft(LeftKnobGlow, centerX - radius);
            Canvas.SetTop(LeftKnobGlow, centerY - radius);

            LeftKnobGlow.Visibility = Visibility.Visible;
        }
    }
}
```