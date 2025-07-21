<Border BorderThickness="3"
        CornerRadius="5"
        Padding="10"
        Margin="10">
    <Border.Style>
        <Style TargetType="Border">
            <Setter Property="BorderBrush" Value="Transparent"/>
            <Style.Triggers>
                <DataTrigger Binding="{Binding IsLeftTempControlled}" Value="True">
                    <Setter Property="BorderBrush" Value="#00FFFF"/> <!-- 亮藍色 -->
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </Border.Style>

    <!-- 溫度顯示與點擊 -->
    <StackPanel Cursor="Hand"
                MouseLeftButtonDown="{Binding SelectLeftTemperatureCommand}">
        <TextBlock Text="{Binding LeftTemperature, StringFormat=0.0°C}"
                   FontSize="140"
                   Foreground="White"
                   HorizontalAlignment="Center"/>
    </StackPanel>
</Border>


<TextBlock Text="{Binding IsLeftTempControlled}" Foreground="Lime"/>