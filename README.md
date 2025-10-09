using System;
using System.Globalization;
using System.Windows.Data;

namespace YourNamespace
{
    public record PatternSelectionArgs(PatternItem? Item, bool IsChecked);

    public class PatternSelectionParamConverter : IMultiValueConverter
    {
        public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
        {
            var item = values.Length > 0 ? values[0] as PatternItem : null;
            var isChecked = values.Length > 1 && values[1] is bool b && b;
            return new PatternSelectionArgs(item, isChecked);
        }

        public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
            => throw new NotImplementedException();
    }
}