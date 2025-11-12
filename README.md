<DataGrid ItemsSource="{Binding IconSlots}"
          AutoGenerateColumns="False"
          HeadersVisibility="Column"
          GridLinesVisibility="None"
          CanUserAddRows="False" CanUserDeleteRows="False"
          RowHeight="44" ColumnHeaderHeight="32">

  <DataGrid.Columns>
    <!-- 1 Icon # -->
    <DataGridTemplateColumn Header="ICON #" Width="120">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                     HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 2 Image Selection（全集 Images） -->
    <DataGridTemplateColumn Header="IMAGE SELECTION" Width="250">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <Button Content="{Binding SelectedImageName}"
                  Padding="10,6" HorizontalAlignment="Stretch"
                  Click="OpenPicker_Click"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 3 SRAM Start Address -->
    <DataGridTemplateColumn Header="SRAM START ADDRESS" Width="160">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding SramStartAddress}" FontFamily="Consolas"
                     HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 3.5 新增：OSD #（顯示 OSD1..OSD30；先用 IconIndex 對應） -->
    <DataGridTemplateColumn Header="OSD #" Width="120">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding IconIndex, StringFormat=OSD{0}}"
                     HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 4 OSD Selection（先用全集 Images，之後一行改來源） -->
    <DataGridTemplateColumn Header="OSD SELECTION" Width="160">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <Button Content="{Binding OsdSelectedImageName}"
                  Padding="10,6" HorizontalAlignment="Stretch"
                  Click="OpenOsdPicker_Click"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 5~7 保持舊欄位 -->
    <DataGridTemplateColumn Header="OSDX_EN" Width="90">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <CheckBox IsChecked="{Binding IsOsdEnabled}" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <DataGridTemplateColumn Header="HPOS" Width="90">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <TextBox Text="{Binding HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                   Width="70" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <DataGridTemplateColumn Header="VPOS" Width="90">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <TextBox Text="{Binding VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                   Width="70" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>
  </DataGrid.Columns>
</DataGrid>