
using System;
using System.Globalization;
using System.Windows.Data;

namespace YourNamespace.Converters // ← ⚠️ 記得換成你實際命名空間
{
    public class VolumeToTopConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            double sliderHeight = 400; // 一定要跟你的 XAML Slider 高度一致
            double min = 0;
            double max = 10;

            double v = System.Convert.ToDouble(value);
            double ratio = (v - min) / (max - min);
            double top = (1 - ratio) * sliderHeight;

            return top;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
}