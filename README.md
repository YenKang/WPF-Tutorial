using System;
using System.Globalization;
using System.Windows;
using System.Windows.Data;

public class PoiSideToAlignmentConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var side = value as string;
        return side == "left" ? HorizontalAlignment.Left : HorizontalAlignment.Right;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}