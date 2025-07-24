<Button Content="Zoom In"
        Command="{Binding ZoomInCommand}"
        Visibility="{Binding DataContext.KnobEnabled,
                     RelativeSource={RelativeSource AncestorType=Window},
                     Converter={StaticResource InvertBoolToVis}}"/>