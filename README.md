<TextBlock Text="{Binding LeftTemperature, StringFormat={}{0:0.0}°C}"
           FontSize="140"
           HorizontalAlignment="Center"
           Cursor="Hand">
    <!-- 點擊時切換控制目標為溫度 -->
    <TextBlock.InputBindings>
        <MouseBinding MouseAction="LeftClick"
                      Command="{Binding SelectLeftTemperatureCommand}" />
    </TextBlock.InputBindings>

    <!-- 高亮邏輯：若目前是控制溫度，則字變 Cyan -->
    <TextBlock.Style>
        <Style TargetType="TextBlock">
            <Setter Property="Foreground" Value="White"/>
            <Style.Triggers>
                <DataTrigger Binding="{Binding IsLeftControllingTemperature}" Value="True">
                    <Setter Property="Foreground" Value="Cyan"/>
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </TextBlock.Style>
</TextBlock>