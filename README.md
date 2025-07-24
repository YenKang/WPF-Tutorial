<Button Content="Zoom In"
        Command="{Binding ZoomInCommand}"
        Visibility="{Binding KnobEnabled, Converter={StaticResource InvertBoolToVis}}"/>