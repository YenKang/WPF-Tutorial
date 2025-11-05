<DataGrid ItemsSource="{Binding Osds}"
          AutoGenerateColumns="False"
          CanUserAddRows="False">
  <DataGrid.Columns>
    <!-- OSD 名稱 -->
    <DataGridTextColumn Header="OSD"
                        Binding="{Binding OsdName}"
                        IsReadOnly="True" Width="110"/>

    <!-- Icon 下拉 -->
    <DataGridTemplateColumn Header="Icon" Width="200">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <ComboBox ItemsSource="{Binding DataContext.Icons, RelativeSource={RelativeSource AncestorType=Window}}"
                    SelectedItem="{Binding SelectedIcon, Mode=TwoWay}"
                    DisplayMemberPath="Name" Width="180"/>
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- Icon 縮圖（獨立一欄） -->
    <DataGridTemplateColumn Header="Icon 縮圖" Width="120">
      <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
          <Image Width="36" Height="36"
                 HorizontalAlignment="Center" VerticalAlignment="Center"
                 Source="{Binding SelectedIcon.Thumbnail}" />
        </DataTemplate>
      </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- Flash address（獨立一欄） -->
    <DataGridTextColumn Header="Flash address (0xAAAAAA)"
                        Binding="{Binding SelectedIcon.FlashStartHex}"
                        Width="*" />
  </DataGrid.Columns>
</DataGrid>