<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        Title="Icon → Image Selection" Width="900" Height="640"
        WindowStartupLocation="CenterOwner">

  <!-- 設計器若找不到 VM，可先註解，改到 .xaml.cs 設定 DataContext -->
  <Window.DataContext>
    <vm:IconToImageMapViewModel/>
  </Window.DataContext>

  <Grid Margin="16">
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- 表頭 -->
    <DockPanel Grid.Row="0" Margin="0,0,0,8">
      <TextBlock Text="Icon#" Width="120" FontWeight="Bold"/>
      <TextBlock Text="Image Selection" Margin="16,0,0,0" FontWeight="Bold"/>
    </DockPanel>

    <!-- 主表 -->
    <DataGrid Grid.Row="1"
              ItemsSource="{Binding IconSlots}"
              AutoGenerateColumns="False"
              CanUserAddRows="False"
              HeadersVisibility="None"
              RowHeaderWidth="0"
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

        <!-- 右：Image Selection（按鈕→圖片牆） -->
        <DataGridTemplateColumn Width="*">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <Button Content="{Binding SelectedImageName}"
                      HorizontalAlignment="Stretch" Padding="10,6"
                      Click="OpenPicker_Click"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

      </DataGrid.Columns>
    </DataGrid>
  </Grid>
</Window>