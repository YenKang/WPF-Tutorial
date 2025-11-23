## UI

```xaml
<TabItem Header="ASCII">
    <Grid Background="Yellow" Margin="8">

        <!-- 兩行：表頭 + 資料列表 -->
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>   <!-- Header -->
            <RowDefinition Height="*"/>      <!-- Data Rows -->
        </Grid.RowDefinitions>

        <!-- 表頭 -->
        <Grid Grid.Row="0" Margin="0,0,0,4">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="2*"/>   <!-- Register Name -->
                <ColumnDefinition Width="*"/>    <!-- Image Select -->
            </Grid.ColumnDefinitions>

            <TextBlock Grid.Column="0"
                       Text="Register Name"
                       FontWeight="Bold"
                       Margin="0,0,0,2"/>

            <TextBlock Grid.Column="1"
                       Text="Image Select"
                       FontWeight="Bold"
                       Margin="8,0,0,2"/>
        </Grid>

        <!-- 資料列表 -->
        <ItemsControl Grid.Row="1"
                      ItemsSource="{Binding AsilSlots}">
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

                        <!-- 右：選圖片按鈕 -->
                        <Button Grid.Column="1"
                                Margin="8,0,0,0"
                                Content="{Binding SelectedImage.Key, FallbackValue=選圖片}"
                                Click="OpenAsilIconPicker_Click"/>
                    </Grid>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

    </Grid>
</TabItem>
```
