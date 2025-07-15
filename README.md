<Slider Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeLevel, Mode=TwoWay}"
        Width="100"
        Height="400"
        HorizontalAlignment="Center">

    <Slider.Resources>
        <Style TargetType="Thumb">
            <!-- 先改 Template -->
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Thumb">
                        <Ellipse Fill="LightSkyBlue"
                                 Stroke="White"
                                 StrokeThickness="2"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
            <!-- 設定圓形尺寸 -->
            <Setter Property="Width" Value="60"/>
            <Setter Property="Height" Value="60"/>
        </Style>
    </Slider.Resources>

</Slider>