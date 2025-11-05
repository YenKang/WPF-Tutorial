<Window x:Class="OSDIconFlashMap.View.IconFlashMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="OSD Icon Flash Map"
        Height="620"
        Width="880"
        WindowStartupLocation="CenterOwner">

  <Grid>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="*"/>
    </Grid.RowDefinitions>

    <!-- 🟦 上方工具列：IC Select -->
    <StackPanel Grid.Row="0"
                Orientation="Horizontal"
                Margin="12,8,12,6">
      <TextBlock Text="IC Select"
                 VerticalAlignment="Center"
                 Margin="0,0,8,0"
                 FontWeight="Bold"/>
      <ComboBox Width="160"
                ItemsSource="{Binding IcOptions}"
                SelectedItem="{Binding SelectedIc, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- 🟩 主表格 -->
    <DataGrid Grid.Row="1"
              ItemsSource="{Binding Osds}"
              AutoGenerateColumns="False"
              CanUserAddRows="False"
              IsReadOnly="False"
              EnableRowVirtualization="False"
              HeadersVisibility="Column"
              GridLinesVisibility="None"
              RowHeaderWidth="0"
              ColumnHeaderHeight="32"
              Margin="12,0,12,12"
              SelectionUnit="Cell">

      <DataGrid.Columns>
        <!-- 1️⃣ OSD 名稱 -->
        <DataGridTextColumn Header="OSD"
                            Width="100"
                            IsReadOnly="True"
                            Binding="{Binding OsdName}" />

        <!-- 2️⃣ ICON（顯示＝文字；編輯＝下拉，提交點＝離開儲存格） -->
        <DataGridTemplateColumn Header="ICON" Width="220">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <TextBlock Text="{Binding SelectedIcon.Name}" VerticalAlignment="Center"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>

          <DataGridTemplateColumn.CellEditingTemplate>
            <DataTemplate>
              <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                        SelectedItem="{Binding SelectedIcon, Mode=TwoWay, UpdateSourceTrigger=LostFocus}"
                        DisplayMemberPath="Name"
                        Width="200"
                        IsDropDownOpen="True"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellEditingTemplate>
        </DataGridTemplateColumn>

        <!-- 3️⃣ Icon 縮圖（目前顯示鍵名/占位文字） -->
        <DataGridTemplateColumn Header="ICON 縮圖" Width="160">
          <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
              <TextBlock Text="{Binding ThumbnailKey}"
                         HorizontalAlignment="Center"
                         VerticalAlignment="Center"
                         Foreground="#666"
                         FontStyle="Italic"/>
            </DataTemplate>
          </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>

        <!-- 4️⃣ Flash Start Address -->
        <DataGridTextColumn Header="FLASH START ADDR (0xAAAAAA)"
                            Width="*">
          <DataGridTextColumn.ElementStyle>
            <Style TargetType="TextBlock">
              <Setter Property="Text" Value="{Binding FlashStartAddrHex}"/>
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