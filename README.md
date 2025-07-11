<StackPanel 
    HorizontalAlignment="Center" 
    VerticalAlignment="Center" 
    Margin="1000,0,0,200">

    <!-- Volume Slider -->
    <Slider
        Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeLevel, Mode=TwoWay}"
        Width="100"
        Height="400"
        Foreground="DodgerBlue"
        Background="#333"/>

    <!-- Volume Text -->
    <TextBlock 
        Text="{Binding VolumeLevel, StringFormat='Volume: {0}'}"
        Foreground="White"
        FontSize="30"
        HorizontalAlignment="Center"
        Margin="0, 10, 0, 0"/>
</StackPanel>