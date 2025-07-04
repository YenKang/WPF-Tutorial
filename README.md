
```xaml
<Grid>
    <Image x:Name="DriverMapImage" Stretch="Uniform" />
</Grid>
```


```csharp
using System;
using System.Windows.Controls;
using System.Windows.Media.Imaging;

namespace NovaCID.Pages
{
    public partial class DriverPage : UserControl
    {
        private readonly string[] _mapPaths = 
        {
            "/DriverMaps/DriverMap_Normal.png",
            "/DriverMaps/DriverMap_ZoomIn+1.png",
            "/DriverMaps/DriverMap_ZoomIn+2.png",
            "/DriverMaps/DriverMap_ZoomOut-1.png",
            "/DriverMaps/DriverMap_ZoomOut-2.png"
        };

        private int _currentIndex = 0;

        public DriverPage()
        {
            InitializeComponent();
            LoadMap();
        }

        private void LoadMap()
        {
            var uri = new Uri(_mapPaths[_currentIndex], UriKind.Relative);
            DriverMapImage.Source = new BitmapImage(uri);
        }

        // 模擬旋鈕右轉（放大）
        public void ZoomIn()
        {
            if (_currentIndex < _mapPaths.Length - 1)
            {
                _currentIndex++;
                LoadMap();
            }
        }

        // 模擬旋鈕左轉（縮小）
        public void ZoomOut()
        {
            if (_currentIndex > 0)
            {
                _currentIndex--;
                LoadMap();
            }
        }
    }
}
```