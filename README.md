<Window x:Class="OSDIconFlashMap.View.IconFlashMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="OSD Icon Flash Map" Height="620" Width="820">
  <Grid Margin="12">
    <DataGrid ItemsSource="{Binding Osds}"
              AutoGenerateColumns="False"
              CanUserAddRows="False"
              RowHeaderWidth="0"
              HeadersVisibility="Column"
              GridLinesVisibility="None"
              ColumnHeaderHeight="32">
      <DataGrid.Columns>

        <!-- 1) OSD -->
        <DataGridTextColumn Header="OSD"
                            Width="100"
                            IsReadOnly="True"
                            Binding="{Binding OsdName}" />

        <!-- 2) ICON（可選） -->
        <DataGridTemplateColumn Header="ICON" Width="220">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                        SelectedItem="{Binding SelectedIcon, Mode=TwoWay}"
                        DisplayMemberPath="Name"
                        Width="200" />
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 3) Icon 縮圖（獨立一欄） -->
        <DataGridTemplateColumn Header="Icon 縮圖" Width="140">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <Border Width="120" Height="40" HorizontalAlignment="Center" VerticalAlignment="Center">
                <Image Source="{Binding SelectedIcon.Thumbnail}"
                       Stretch="Uniform"
                       HorizontalAlignment="Center" VerticalAlignment="Center"/>
              </Border>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 4) Flash address（獨立一欄） -->
        <DataGridTextColumn Header="Flash address (0xAAAAAA)"
                            Width="*">
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