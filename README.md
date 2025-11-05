<Window x:Class="OSDIconFlashMap.View.IconFlashMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="OSD Icon Flash Map" Height="620" Width="860">
  <Grid>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- 頂部工具列：IC Select -->
    <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="12,8,12,4">
      <TextBlock Text="IC Select" VerticalAlignment="Center" Margin="0,0,8,0" FontWeight="Bold"/>
      <ComboBox Width="160"
                ItemsSource="{Binding IcOptions}"
                SelectedItem="{Binding SelectedIc, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
      <!-- 需要時可放更多控制項 -->
    </StackPanel>

    <!-- 主表格 -->
    <DataGrid Grid.Row="1"
              ItemsSource="{Binding Osds}"
              AutoGenerateColumns="False"
              CanUserAddRows="False"
              RowHeaderWidth="0"
              HeadersVisibility="Column"
              GridLinesVisibility="None"
              ColumnHeaderHeight="32"
              Margin="12,0,12,12">
      <DataGrid.Columns>
        <!-- 1) OSD -->
        <DataGridTextColumn Header="OSD" Width="100" IsReadOnly="True" Binding="{Binding OsdName}" />

        <!-- 2) ICON（可選） -->
        <DataGridTemplateColumn Header="ICON" Width="220">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                        SelectedItem="{Binding SelectedIcon, Mode=TwoWay}"
                        DisplayMemberPath="Name" Width="200"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 3) Icon 縮圖（目前顯示鍵名/占位） -->
        <DataGridTemplateColumn Header="ICON 縮圖" Width="160">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <TextBlock Text="{Binding SelectedIcon.ThumbnailKey}"
                         HorizontalAlignment="Center" VerticalAlignment="Center"
                         Foreground="#666" FontStyle="Italic"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 4) Flash address -->
        <DataGridTextColumn Header="FLASH ADDRESS (0xAAAAAA)" Width="*">
          <DataGridTextColumn.ElementStyle>
            <Style TargetType="TextBlock">
              <Setter Property="Text" Value="{Binding SelectedIcon.FlashStartHex}"/>
              <Setter Property="VerticalAlignment" Value="Center"/>
              <Setter Property="TextAlignment" Value="Right"/>
              <Setter Property="Margin" Value="0,0,12,0"/>
              <Setter Property="FontFamily" Value="Consolas"/>
            </Style>
          </DataGridTextColumn.ElementStyle>
        </DataGridTextColumn>

      </DataGrid.Columns>
    </DataGrid>
  </Grid>
</Window>