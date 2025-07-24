<TextBlock Text="{Binding DataContext.KnobEnabled,
                  RelativeSource={RelativeSource AncestorType=Window}}"
           Foreground="Red"/>


```charp
public class InvertedBooleanToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if (value is bool b)
            return b ? Visibility.Collapsed : Visibility.Visible;
        return Visibility.Collapsed;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        => DependencyProperty.UnsetValue;
}
```

<local:InvertedBooleanToVisibilityConverter x:Key="InvertBoolToVis"/>