<ItemsControl ItemsSource="{Binding FanBarIndexList}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <!-- 這一列的 DataContext == 當前 index (0..N-1) -->
            <Rectangle Height="30"
                       Width="300"
                       Margin="25"
                       Cursor="Hand">
                <Rectangle.Fill>
                    <!-- 你的 converter 完全不動 -->
                    <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                        <!-- currentLevel（1-based） -->
                        <Binding Path="DataContext.LeftFanLevel"
                                 RelativeSource="{RelativeSource AncestorType=UserControl}" />
                        <!-- 這條的 index（0-based） -->
                        <Binding />
                    </MultiBinding>
                </Rectangle.Fill>

                <!-- ✅ 這裡把 LeftClick → Command -->
                <Rectangle.InputBindings>
                    <MouseBinding MouseAction="LeftClick"
                                  Command="{Binding DataContext.SetLeftFanLevelCmd,
                                                    RelativeSource={RelativeSource AncestorType=UserControl}}"
                                  CommandParameter="{Binding}" />
                </Rectangle.InputBindings>
            </Rectangle>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>