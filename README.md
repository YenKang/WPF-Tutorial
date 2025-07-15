<Slider Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeLevel, Mode=TwoWay}"
        Width="120"  <!-- 整個 Slider 外框更寬 -->
        Height="400"
        HorizontalAlignment="Center">

    <Slider.Resources>
        <!-- Thumb 樣式 -->
        <Style TargetType="Thumb">
            <Setter Property="Width" Value="60"/>
            <Setter Property="Height" Value="60"/>
            <Setter Property="Background" Value="LightSkyBlue"/>
            <Setter Property="BorderBrush" Value="White"/>
            <Setter Property="BorderThickness" Value="2"/>
        </Style>

        <!-- Track 樣式 -->
        <Style TargetType="Track">
            <Setter Property="Height" Value="400"/>
            <Setter Property="Width" Value="20"/> <!-- ✅ 這裡把藍色條變粗 -->
        </Style>
    </Slider.Resources>

</Slider>