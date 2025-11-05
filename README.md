<DataGridTemplateColumn Header="Icon 縮圖" Width="140">
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <TextBlock Text="{Binding SelectedIcon.ThumbnailKey}"
                 VerticalAlignment="Center" HorizontalAlignment="Center"
                 FontStyle="Italic" Foreground="#666"/>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>