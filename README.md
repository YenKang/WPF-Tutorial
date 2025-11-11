<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Icon → Image Selection"
        Width="900" Height="620"
        WindowStartupLocation="CenterOwner">

    <Grid Margin="16">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>   <!-- 工具列 -->
            <RowDefinition Height="*"/>      <!-- 主表格 -->
        </Grid.RowDefinitions>

        <!-- ✅ 工具列：Save / Load INI -->
        <DockPanel Grid.Row="0" Margin="0,0,0,8">
            <StackPanel Orientation="Horizontal" DockPanel.Dock="Left" VerticalAlignment="Center">
                <Button Content="Save INI" Margin="0,0,8,0" Padding="10,4" Click="OnSaveIni"/>
                <Button Content="Load INI"               Padding="10,4" Click="OnLoadIni"/>
            </StackPanel>

            <StackPanel Orientation="Horizontal" DockPanel.Dock="Right">
                <TextBlock Text="Icon#" Width="120" FontWeight="Bold"/>
                <TextBlock Text="Image Selection" Margin="16,0,0,0" FontWeight="Bold"/>
            </StackPanel>
        </DockPanel>

        <!-- ✅ 主表格 -->
        <DataGrid Grid.Row="1"
                  ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  HeadersVisibility="None"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  CanUserResizeRows="False"
                  IsReadOnly="True"
                  GridLinesVisibility="None"
                  RowHeight="36">

            <DataGrid.Columns>
                <!-- 左欄：Icon# -->
                <DataGridTemplateColumn Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Border Padding="8"
                                    BorderBrush="#DDD"
                                    BorderThickness="1"
                                    CornerRadius="4"
                                    Margin="0,0,8,0">
                                <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                                           VerticalAlignment="Center"
                                           HorizontalAlignment="Center"/>
                            </Border>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 右欄：Image Selection（按鈕） -->
                <DataGridTemplateColumn Width="250">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="{Binding SelectedImageName, Mode=OneWay}"
                                    HorizontalAlignment="Stretch"
                                    Padding="10,6"
                                    Click="OnOpenPickerClick"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>