using System;
using System.Collections.Generic;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media.Imaging;

namespace 你的命名空間
{
    public partial class DriverPage : Page
    {
        private readonly List<string> _mapImages = new List<string>
        {
            "/Assets/Maps/DriverMap1.png",
            "/Assets/Maps/DriverMap2.png",
            "/Assets/Maps/DriverMap3.png",
            "/Assets/Maps/DriverMap4.png",
            "/Assets/Maps/DriverMap5.png"
        };

        private int _currentZoomIndex = 0;

        public DriverPage()
        {
            InitializeComponent();
            ShowMap(_currentZoomIndex);
        }

        private void ShowMap(int index)
        {
            if (index < 0 || index >= _mapImages.Count) return;

            DriverMapImage.Source = new BitmapImage(new Uri(_mapImages[index], UriKind.Relative));
        }

        private void ZoomIn_Click(object sender, RoutedEventArgs e)
        {
            if (_currentZoomIndex < _mapImages.Count - 1)
            {
                _currentZoomIndex++;
                ShowMap(_currentZoomIndex);
            }
        }

        private void ZoomOut_Click(object sender, RoutedEventArgs e)
        {
            if (_currentZoomIndex > 0)
            {
                _currentZoomIndex--;
                ShowMap(_currentZoomIndex);
            }
        }
    }
}