<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Icon → Image Selection"
        Width="920" Height="660"
        WindowStartupLocation="CenterOwner">

    <!-- 不在這裡綁 DataContext，改由 .xaml.cs 建構子統一處理，避免被外部覆蓋 -->

    <Grid Margin="16">
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <!-- 資料格（七欄） -->
        <DataGrid Grid.Row="0"
                  ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  HeadersVisibility="Column"
                  GridLinesVisibility="None"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  RowHeight="44"
                  ColumnHeaderHeight="32"
                  FontSize="13">

            <DataGrid.Columns>
                <!-- 1 Icon # -->
                <DataGridTemplateColumn Header="Icon #" Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                                       HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 2 Image Selection（按鈕呼叫 OpenPicker_Click） -->
                <DataGridTemplateColumn Header="Image Selection" Width="250">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="{Binding SelectedImageName}"
                                    Padding="10,6" HorizontalAlignment="Stretch"
                                    Click="OpenPicker_Click"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 3 SRAM Start Address（唯讀文字） -->
                <DataGridTemplateColumn Header="SRAM Start Address" Width="160">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding SramStartAddress}"
                                       FontFamily="Consolas"
                                       HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 4 OSD Selection（1..30） -->
                <DataGridTemplateColumn Header="OSD Selection" Width="140">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <ComboBox Width="120"
                                      SelectedValuePath="."
                                      SelectedValue="{Binding OsdTargetIndex, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
                                <ComboBox.ItemsSource>
                                    <!-- 從 Window.DataContext 取 OsdOptions -->
                                    <Binding Path="DataContext.OsdOptions"
                                             RelativeSource="{RelativeSource AncestorType=Window}"/>
                                </ComboBox.ItemsSource>
                            </ComboBox>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 5 OSDx_EN -->
                <DataGridTemplateColumn Header="OSDx_EN" Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <CheckBox IsChecked="{Binding IsOsdEnabled, Mode=TwoWay}"
                                      HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 6 HPos -->
                <DataGridTemplateColumn Header="HPos" Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBox Text="{Binding HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                     Width="70" HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 7 VPos -->
                <DataGridTemplateColumn Header="VPos" Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBox Text="{Binding VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                     Width="70" HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>

        <!-- OK / Cancel（用來輸出 ResultMap；不需要可移除） -->
        <StackPanel Grid.Row="1" Orientation="Horizontal"
                    HorizontalAlignment="Right" Margin="0,12,0,0">
            <Button Content="OK" Width="100" Height="32" Click="ConfirmAndClose"/>
            <Button Content="Cancel" Width="100" Height="32" Margin="8,0,0,0" IsCancel="True"/>
        </StackPanel>
    </Grid>
</Window>