using System;
using System.Globalization;
using System.Windows.Data;
using System.Windows.Media;

namespace YourProjectNamespace.Converters
{
    public class KnobRoleToColorConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is KnobRole role && role != KnobRole.None)
            {
                return Colors.DeepSkyBlue; // ✅ 可改成想要的 Glow 顏色
            }
            return Colors.Transparent;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
}


=========

using System;
using System.Globalization;
using System.Windows;
using System.Windows.Data;

namespace YourProjectNamespace.Converters
{
    public class RoleToOpacityConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            return (value is KnobRole role && role != KnobRole.None) ? 1.0 : 0.0;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
}