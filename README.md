<StackPanel Cursor="Hand">
    <StackPanel.InputBindings>
        <MouseBinding Gesture="LeftClick"
                      Command="{Binding SelectLeftTemperatureCommand}" />
    </StackPanel.InputBindings>

    <TextBlock Text="{Binding LeftTemperature, StringFormat=0.0°C}"
               FontSize="140"
               Foreground="White"
               HorizontalAlignment="Center"/>
</StackPanel>