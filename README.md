<!-- 左邊風量區（有外框） -->
<Border BorderThickness="3" CornerRadius="10" Margin="10">
    <Border.Style>
        <Style TargetType="Border">
            <Setter Property="BorderBrush" Value="Transparent"/>
            <Style.Triggers>
                <DataTrigger Binding="{Binding LeftControlTarget}" Value="Fan">
                    <Setter Property="BorderBrush" Value="#00FFFF"/>
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </Border.Style>

    <StackPanel Orientation="Vertical" VerticalAlignment="Bottom">
        <ItemsControl ItemsSource="{Binding FanBarIndexList}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Rectangle Height="30" Width="300" Margin="30"
                               Cursor="Hand" MouseLeftButtonDown="LeftFanBar_Click">
                        <Rectangle.Fill>
                            <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                                <Binding Path="DataContext.LeftFanLevel" RelativeSource="{RelativeSource AncestorType=UserControl}" />
                                <Binding />
                            </MultiBinding>
                        </Rectangle.Fill>
                    </Rectangle>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </StackPanel>
</Border>


<!-- 左邊溫度區（有外框） -->
<Border BorderThickness="3" CornerRadius="10" Margin="10">
    <Border.Style>
        <Style TargetType="Border">
            <Setter Property="BorderBrush" Value="Transparent"/>
            <Style.Triggers>
                <DataTrigger Binding="{Binding LeftControlTarget}" Value="Temp">
                    <Setter Property="BorderBrush" Value="#00FFFF"/>
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </Border.Style>

    <!-- 原本溫度數值顯示 -->
    <TextBlock Text="{Binding LeftTemperature, StringFormat=0.0°C}"
               FontSize="140"
               Foreground="White"
               HorizontalAlignment="Center"/>
</Border>