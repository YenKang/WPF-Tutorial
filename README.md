<!-- Pattern Total：1 ~ 22 (UpDown) -->
<StackPanel Orientation="Horizontal" Margin="0,0,0,12">
    <TextBlock Text="Pattern Total"
               VerticalAlignment="Center"
               Margin="0,0,12,0"/>
    <xctk:IntegerUpDown Width="90"
                        Minimum="1"
                        Maximum="22"
                        Value="{Binding Total,
                                        Mode=TwoWay,
                                        UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>

＝＝＝＝＝

<!-- Pattern Orders：固定 22 列，內容是數字 (1~22) -->
<ItemsControl ItemsSource="{Binding Orders}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
                <!-- 顯示第幾筆：1,2,3... -->
                <TextBlock Text="{Binding DisplayNo}"
                           VerticalAlignment="Center"
                           Width="72"/>

                <!-- 純數字選擇：1~22 -->
                <ComboBox Width="96"
                          ItemsSource="{Binding DataContext.PatternNumberOptions,
                                                RelativeSource={RelativeSource AncestorType=GroupBox}}"
                          SelectedItem="{Binding SelectedPatternNumber,
                                                 Mode=TwoWay,
                                                 UpdateSourceTrigger=PropertyChanged}"/>
            </StackPanel>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
