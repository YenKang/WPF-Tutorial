using System;
using System.Globalization;
using System.Windows.Data;

namespace OSDIconFlashMap.Converters
{
    public class BoolTo01Converter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
            => (value is bool b && b) ? "1" : "0";

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
            => (value?.ToString() == "1");
    }
}