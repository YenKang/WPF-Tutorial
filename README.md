<Grid x:Name="PoiCard"
      Width="200"
      Height="120"
      Background="#AA222222"
      HorizontalAlignment="Center"
      VerticalAlignment="Top"
      Margin="0,50,0,0"
      Visibility="{Binding IsPoiVisible, Converter={StaticResource BoolToVisibilityConverter}}">
    <TextBlock Text="POI 資訊卡片"
               Foreground="White"
               FontSize="18"
               HorizontalAlignment="Center"
               VerticalAlignment="Center"/>
</Grid>

```csharp
using System;
using System.Globalization;
using System.Windows;
using System.Windows.Data;

public class BoolToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool && (bool)value)
            return Visibility.Visible;
        else
            return Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is Visibility && (Visibility)value == Visibility.Visible)
            return true;
        else
            return false;
    }
}
```