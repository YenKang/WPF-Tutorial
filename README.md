<DataGridTemplateColumn Header="ICON" Width="220">
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                SelectedItem="{Binding SelectedIcon, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                DisplayMemberPath="Name" Width="200"/>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
  <DataGridTemplateColumn.CellEditingTemplate>
    <DataTemplate>
      <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                SelectedItem="{Binding SelectedIcon, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                DisplayMemberPath="Name" Width="200" IsDropDownOpen="True"/>
    </DataTemplate>
  </DataGridTemplateColumn.CellEditingTemplate>
</DataGridTemplateColumn>