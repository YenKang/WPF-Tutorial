<ItemsControl ItemsSource="{Binding FanBarIndexList}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Border BorderThickness="3" CornerRadius="5" Padding="2" Margin="4">
                <Border.Style>
                    <Style TargetType="Border">
                        <Setter Property="BorderBrush" Value="Transparent"/>
                        <Style.Triggers>
                            <DataTrigger Binding="{Binding DataContext.IsLeftFanControlled, RelativeSource={RelativeSource AncestorType=UserControl}}" 
                                         Value="True">
                                <Setter Property="BorderBrush" Value="#00FFFF"/>
                            </DataTrigger>
                        </Style.Triggers>
                    </Style>
                </Border.Style>

                <!-- 原始風量 Rectangle -->
                <Rectangle Height="30" Width="300" Cursor="Hand" MouseLeftButtonDown="LeftFanBar_Click">
                    <Rectangle.Fill>
                        <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                            <!-- LeftFanLevel -->
                            <Binding Path="DataContext.LeftFanLevel" RelativeSource="{RelativeSource AncestorType=UserControl}" />
                            <!-- index -->
                            <Binding />
                        </MultiBinding>
                    </Rectangle.Fill>
                </Rectangle>
            </Border>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>