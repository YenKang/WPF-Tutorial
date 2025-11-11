<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        Title="Icon → Image Selection"
        Width="900" Height="620"
        WindowStartupLocation="CenterOwner">

    <Grid Margin="16">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- 工具列：IC Select + Save/Load -->
        <DockPanel Grid.Row="0" Margin="0,0,0,8">

            <StackPanel Orientation="Horizontal" DockPanel.Dock="Left" VerticalAlignment="Center">
                <TextBlock Text="IC Select:" Margin="0,0,8,0" VerticalAlignment="Center" FontWeight="Bold"/>
                <!-- 如果 VM 有 enum IcProfile { Primary, L1, L2, L3 } -->
                <ComboBox Width="140"
                          SelectedItem="{Binding SelectedProfile, Mode=TwoWay}"
                          VerticalAlignment="Center">
                    <ComboBox.ItemsSource>
                        <x:Array Type="{x:Type vm:IcProfile}">
                            <vm:IcProfile>Primary</vm:IcProfile>
                            <vm:IcProfile>L1</vm:IcProfile>
                            <vm:IcProfile>L2</vm:IcProfile>
                            <vm:IcProfile>L3</vm:IcProfile>
                        </x:Array>
                    </ComboBox.ItemsSource>
                </ComboBox>

                <Button Content="Save INI" Margin="12,0,0,0" Padding="10,4" Click="OnSaveIni"/>
                <Button Content="Load INI" Margin="8,0,0,0"  Padding="10,4" Click="OnLoadIni"/>
            </StackPanel>

            <!-- 右側欄位標題（沿用你原本樣式） -->
            <StackPanel Orientation="Horizontal" DockPanel.Dock="Right">
                <TextBlock Text="Icon#" Width="120" FontWeight="Bold"/>
                <TextBlock Text="Image Selection" Margin="16,0,0,0" FontWeight="Bold"/>
            </StackPanel>
        </DockPanel>

        <!-- 表格 -->
        <DataGrid Grid.Row="1"
                  ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  CanUserAddRows="False"
                  HeadersVisibility="None"
                  BorderThickness="0"
                  RowHeight="44"
                  GridLinesVisibility="None">

            <DataGrid.Columns>

                <!-- 左：Icon# -->
                <DataGridTemplateColumn Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Border Padding="8" BorderBrush="#DDD" BorderThickness="1" CornerRadius="4">
                                <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                                           VerticalAlignment="Center"/>
                            </Border>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 右：Image Selection（按鈕→開圖片牆） -->
                <DataGridTemplateColumn Width="250">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="{Binding SelectedImageName, Mode=OneWay}"
                                    HorizontalAlignment="Stretch"
                                    Padding="10,6"
                                    Click="OpenPicker_Click"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>