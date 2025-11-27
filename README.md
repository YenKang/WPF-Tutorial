<TabItem Header="CSP">
    <Grid Margin="8">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />  <!-- Header -->
            <RowDefinition Height="*" />     <!-- 資料列 -->
        </Grid.RowDefinitions>

        <!-- 🔹 Header 列 -->
        <Grid Grid.Row="0" Margin="0,0,0,4">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="2*" />  <!-- CSP Register -->
                <ColumnDefinition Width="*" />   <!-- Value -->
                <ColumnDefinition Width="2*" />  <!-- UI -->
            </Grid.ColumnDefinitions>

            <TextBlock Grid.Column="0"
                       Text="CSP Register"
                       FontWeight="Bold"
                       Margin="4,0" />

            <TextBlock Grid.Column="1"
                       Text="Value"
                       FontWeight="Bold"
                       Margin="4,0" />

            <TextBlock Grid.Column="2"
                       Text="UI"
                       FontWeight="Bold"
                       Margin="4,0" />
        </Grid>

        <!-- 🔹 資料列（之後你可以綁 CspSlots） -->
        <ScrollViewer Grid.Row="1"
                      VerticalScrollBarVisibility="Auto">
            <ItemsControl ItemsSource="{Binding CspSlots}">
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Grid Margin="0,2">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="2*" />
                                <ColumnDefinition Width="*" />
                                <ColumnDefinition Width="2*" />
                            </Grid.ColumnDefinitions>

                            <!-- CSP Register 名稱 -->
                            <TextBlock Grid.Column="0"
                                       Text="{Binding RegName}"
                                       Margin="4,0" />

                            <!-- Value，可改 TextBlock 或其他 -->
                            <TextBox Grid.Column="1"
                                     Text="{Binding Value}"
                                     Margin="4,0"
                                     HorizontalContentAlignment="Center" />

                            <!-- UI 區塊，先用 TextBlock 佔位 -->
                            <!-- 之後可以改成 ComboBox / CheckBox / 自訂控件 -->
                            <ContentControl Grid.Column="2"
                                            Margin="4,0"
                                            Content="{Binding UiElement}">
                                <!-- 若還沒做 UiElement，也可以暫時用 TextBlock：
                                <TextBlock Text="{Binding UiLabel}" />
                                -->
                            </ContentControl>
                        </Grid>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </ScrollViewer>
    </Grid>
</TabItem>
