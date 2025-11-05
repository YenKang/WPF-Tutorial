<Window x:Class="OSDIconFlashMap.View.IconFlashMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="OSD Icon Flash Map" Height="620" Width="780">
  <Grid Margin="12">
    <DataGrid ItemsSource="{Binding Osds}"
              AutoGenerateColumns="False"
              CanUserAddRows="False">
      <DataGrid.Columns>
        <!-- 左欄：OSD_1..OSD_40 -->
        <DataGridTextColumn Header="OSD"
                            Binding="{Binding OsdName}"
                            IsReadOnly="True" Width="110"/>

        <!-- 中欄：icon_1..icon_N -->
        <DataGridTemplateColumn Header="Icon" Width="220">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                        SelectedItem="{Binding SelectedIcon, Mode=TwoWay}"
                        DisplayMemberPath="Name" Width="200"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 右欄：縮圖 + Flash address -->
        <DataGridTemplateColumn Header="Icon 縮圖 / Flash address" Width="*">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <StackPanel Orientation="Horizontal">
                <Image Width="36" Height="36" Source="{Binding SelectedIcon.Thumbnail}" Margin="4"/>
                <StackPanel>
                  <TextBlock Text="{Binding SelectedIcon.Name}"/>
                  <TextBlock Text="{Binding SelectedIcon.FlashStartHex}" FontSize="11"/>
                </StackPanel>
              </StackPanel>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>
      </DataGrid.Columns>
    </DataGrid>
  </Grid>
</Window>