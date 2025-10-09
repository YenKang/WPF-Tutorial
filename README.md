public class BoolToBrushConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        bool isSelected = value is bool b && b;
        return isSelected ? new SolidColorBrush(Color.FromRgb(33,150,243)) : new SolidColorBrush(Color.FromRgb(50,50,50));
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture) 
        => throw new NotImplementedException();
}