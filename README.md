```xml
<StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">

    <!-- Fan Icon -->
    <Image
        Source="/Assets/fan_icon.png" <!-- 這裡換成你剛剛生成的 icon 路徑 -->
        Width="60"
        Height="60"
        HorizontalAlignment="Center"
        Margin="0,0,0,20"/>

    <!-- Fan Level Slider -->
    <Slider
        Orientation="Vertical"
        Minimum="0"
        Maximum="5"
        Value="{Binding RightFanLevel}"
        Width="80"
        Height="300"
        HorizontalAlignment="Center"
        Margin="0,0,0,20"/>

    <!-- Temperature Text -->
    <TextBlock
        Text="{Binding RightTemperature, StringFormat='{}{0:0.0}°C'}"
        Foreground="White"
        FontSize="36"
        HorizontalAlignment="Center"/>
</StackPanel>
```