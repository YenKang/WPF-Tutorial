<StackPanel HorizontalAlignment="Right" VerticalAlignment="Center" Margin="0,0,80,0">
    <Slider Orientation="Vertical" Minimum="0" Maximum="100"
            Value="{Binding VolumeValue, Mode=TwoWay}" 
            Width="40" Height="300" 
            Foreground="DodgerBlue" Background="#333"/>

    <TextBlock Text="🔊" Foreground="White" FontSize="26" 
               HorizontalAlignment="Center" Margin="0,10,0,0"/>
</StackPanel>