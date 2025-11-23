<!-- Pattern Orders -->
<ItemsControl ItemsSource="{Binding AutoRunVM.Orders}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Margin="4">

                <!-- 顯示序號（你最新的 Index 屬性） -->
                <TextBlock Text="{Binding Index}"
                           VerticalAlignment="Center"
                           Width="40"/>

                <!-- UpDown 綁定 Value（你最新的 Value 屬性） -->
                <xctk:IntegerUpDown Width="80"
                                    Minimum="0"
                                    Maximum="255"
                                    Value="{Binding Value,
                                                    Mode=TwoWay,
                                                    UpdateSourceTrigger=PropertyChanged}"/>
            </StackPanel>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
