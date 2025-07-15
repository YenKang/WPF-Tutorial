<Slider Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeLevel, Mode=TwoWay}"
        Width="100"
        Height="400">

    <Slider.Resources>
        <Style TargetType="Thumb">
            <Setter Property="Width" Value="60"/>
            <Setter Property="Height" Value="60"/>
            <Setter Property="Background" Value="LightSkyBlue"/> <!-- 可自行修改顏色 -->
            <Setter Property="BorderBrush" Value="White"/>
            <Setter Property="BorderThickness" Value="2"/>
        </Style>
    </Slider.Resources>

</Slider>