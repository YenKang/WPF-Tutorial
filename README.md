<DataGrid x:Name="dg"
          ItemsSource="{Binding IconSlots}"
          AutoGenerateColumns="False"
          HeadersVisibility="Column"
          CanUserAddRows="False"
          CanUserDeleteRows="False"
          CanUserResizeRows="False"
          IsReadOnly="True"
          GridLinesVisibility="Horizontal">
    <DataGrid.Columns>
        <DataGridTextColumn Header="Icon#"
                            Binding="{Binding IconIndex}"
                            Width="100" />

        <DataGridTextColumn Header="Image Selection"
                            Binding="{Binding SelectedImageName}" />

        <DataGridTemplateColumn Header="操作">
            ...
        </DataGridTemplateColumn>
    </DataGrid.Columns>
</DataGrid>