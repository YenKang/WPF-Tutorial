<!-- Left Fan 控制區塊 -->
<Border BorderThickness="3"
        CornerRadius="10"
        Margin="20"
        Cursor="Hand">
    
    <!-- 點擊整塊邊框 → 切換控制目標為 Fan -->
    <Border.InputBindings>
        <MouseBinding MouseAction="LeftClick" Command="{Binding SelectLeftFanCommand}" />
    </Border.InputBindings>

    <!-- 高亮外框顏色變更 -->
    <Border.Style>
        <Style TargetType="Border">
            <Setter Property="BorderBrush" Value="Transparent"/>
            <Style.Triggers>
                <DataTrigger Binding="{Binding IsLeftControllingFan}" Value="True">
                    <Setter Property="BorderBrush" Value="Cyan"/>
                </DataTrigger>
            </Style.Triggers>
        </Style>
    </Border.Style>

    <!-- FanBar 條列圖（維持原結構） -->
    <StackPanel Orientation="Vertical" VerticalAlignment="Bottom">
        <ItemsControl ItemsSource="{Binding FanBarIndexList}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Rectangle Height="30"
                               Width="300"
                               Margin="5"
                               Cursor="Hand"
                               MouseLeftButtonDown="LeftFanBar_Click">
                        <Rectangle.Fill>
                            <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                                <!-- LeftFanLevel (目前段數) -->
                                <Binding Path="DataContext.LeftFanLevel" RelativeSource="{RelativeSource AncestorType=UserControl}" />
                                <!-- index (目前這條 bar 的索引) -->
                                <Binding />
                            </MultiBinding>
                        </Rectangle.Fill>
                    </Rectangle>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </StackPanel>
</Border>