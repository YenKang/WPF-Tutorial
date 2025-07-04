
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
    public partial class DriverPage : Page
    {
        // 所有地圖圖片路徑（已去掉 DriverMap_ 前綴）
        private readonly string[] _mapPaths =
        {
            "pack://application:,,,/NovaCID;component/Assets/DriverMaps/Normal.png",
            "pack://application:,,,/NovaCID;component/Assets/DriverMaps/ZoomIn+1.png",
            "pack://application:,,,/NovaCID;component/Assets/DriverMaps/ZoomIn+2.png",
            "pack://application:,,,/NovaCID;component/Assets/DriverMaps/ZoomOut-1.png",
            "pack://application:,,,/NovaCID;component/Assets/DriverMaps/ZoomOut-2.png"
        };

        private int _currentMapIndex = 0;

        public DriverPage()
        {
            InitializeComponent();

            // 預設先載入 Normal
            _currentMapIndex = 0;
            LoadMap();
        }

        private void LoadMap()
        {
            if (_currentMapIndex < 0) _currentMapIndex = 0;
            if (_currentMapIndex >= _mapPaths.Length) _currentMapIndex = _mapPaths.Length - 1;

            DriverMapImage.Source = new BitmapImage(new Uri(_mapPaths[_currentMapIndex], UriKind.Absolute));
        }

        private void ZoomIn_Click(object sender, RoutedEventArgs e)
        {
            if (_currentMapIndex < 2) // 最多放到 ZoomIn+2
            {
                _currentMapIndex++;
                LoadMap();
            }
        }

        private void ZoomOut_Click(object sender, RoutedEventArgs e)
        {
            if (_currentMapIndex > 0) // 最多縮小到 ZoomOut-2
            {
                _currentMapIndex--;
                LoadMap();
            }
        }
    }
}
```