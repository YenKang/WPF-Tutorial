bool flag = value is bool b && b;
        return flag ? Visibility.Collapsed : Visibility.Visible;


<Button Content="Toggle POI Card"
        Command="{Binding ToggleCardCommand}"
        Visibility="{Binding KnobEnabled, Converter={StaticResource InvertBoolToVis}}"/>


<TextBlock Text="{Binding KnobEnabled}"
           Foreground="Lime"
           FontSize="16"/>