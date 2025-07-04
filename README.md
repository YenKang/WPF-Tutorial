
```xaml
<Grid>
    <Image x:Name="DriverMapImage"
           Width="1600"
           Height="900"
           Source="/NovaCID;component/Assets/DriverMaps/Normal.png"
           Stretch="Uniform"/>
</Grid>
```


```csharp
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media.Imaging;

namespace NovaCID.Pages
{
    public partial class DriverPage : UserControl
    {
        // 定義 Zoom Level 列舉
        private enum MapZoomLevel
        {
            ZoomOut2 = 0,
            ZoomOut1 = 1,
            Normal   = 2,
            ZoomIn1  = 3,
            ZoomIn2  = 4
        }

        private MapZoomLevel _currentZoomLevel = MapZoomLevel.Normal;

        public DriverPage()
        {
            InitializeComponent();
            LoadMap();
        }

        private void LoadMap()
        {
            string imageName = _currentZoomLevel switch
            {
                MapZoomLevel.ZoomOut2 => "ZoomOut-2.png",
                MapZoomLevel.ZoomOut1 => "ZoomOut-1.png",
                MapZoomLevel.Normal   => "Normal.png",
                MapZoomLevel.ZoomIn1  => "ZoomIn+1.png",
                MapZoomLevel.ZoomIn2  => "ZoomIn+2.png",
                _ => "Normal.png"
            };

            DriverMapImage.Source = new BitmapImage(new Uri($"/NovaCID;component/Assets/DriverMaps/{imageName}", UriKind.Relative));
        }

        private void ZoomIn_Click(object sender, RoutedEventArgs e)
        {
            if (_currentZoomLevel < MapZoomLevel.ZoomIn2)
            {
                _currentZoomLevel++;
                LoadMap();
            }
        }

        private void ZoomOut_Click(object sender, RoutedEventArgs e)
        {
            if (_currentZoomLevel > MapZoomLevel.ZoomOut2)
            {
                _currentZoomLevel--;
                LoadMap();
            }
        }
    }
}
```