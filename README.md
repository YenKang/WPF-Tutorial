<TabItem Header="ASCII">
    <Grid Background="Yellow" Margin="8">
        <StackPanel>

            <!-- Debug：顯示 AsciiSlots.Count -->
            <TextBlock Text="AsciiSlots.Count = " />
            <TextBlock Text="{Binding AsciiSlots.Count}"
                       FontSize="16"
                       FontWeight="Bold"
                       Margin="0,0,0,8"/>

            <!-- 真正的列表 -->
            <ItemsControl ItemsSource="{Binding AsciiSlots}">
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Grid Margin="0,4">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="2*"/>
                                <ColumnDefinition Width="*"/>
                            </Grid.ColumnDefinitions>

                            <!-- 左：RegName -->
                            <TextBlock Grid.Column="0"
                                       Text="{Binding RegName}"
                                       VerticalAlignment="Center"/>

                            <!-- 右：按鈕 -->
                            <Button Grid.Column="1"
                                    Margin="8,0,0,0"
                                    Content="{Binding SelectedImage.Key, FallbackValue=選圖片}"
                                    Click="OpenAsilIconPicker_Click"/>
                        </Grid>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>

        </StackPanel>
    </Grid>
</TabItem>
