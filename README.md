<!-- OSDx_EN -->
<DataGridTemplateColumn Header="OSDXEN" Width="90">
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <CheckBox IsChecked="{Binding IsOsdEnabled, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>

<!-- TTALEEN -->
<DataGridTemplateColumn Header="TTALEEN" Width="90">
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <CheckBox IsChecked="{Binding IsTtalEnabled, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>