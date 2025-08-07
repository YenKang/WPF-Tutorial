<ItemsControl ItemsSource="{Binding FanBarIndexList}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Rectangle Width="60" Height="12" Margin="2"
                       Cursor="Hand">
                <Rectangle.InputBindings>
                    <MouseBinding Gesture="LeftClick"
                                  Command="{Binding DataContext.SetLeftFanLevelCommand, RelativeSource={RelativeSource AncestorType=UserControl}}"
                                  CommandParameter="{Binding}" />
                </Rectangle.InputBindings>

                <Rectangle.Fill>
                    <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                        <Binding Path="DataContext.LeftFanLevel"
                                 RelativeSource="{RelativeSource AncestorType=UserControl}" />
                        <Binding Path="." />
                    </MultiBinding>
                </Rectangle.Fill>
            </Rectangle>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>