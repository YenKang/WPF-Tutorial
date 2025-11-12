<DataGrid ItemsSource="{Binding IconSlots}"
          AutoGenerateColumns="False"
          CanUserAddRows="False"
          RowHeight="44"
          GridLinesVisibility="None">

    <!-- 1. Icon #N（唯讀） -->
    <DataGridTemplateColumn Header="Icon #N" Width="90">
        <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}" VerticalAlignment="Center"/>
            </DataTemplate>
        </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 2. Image Selection（按鈕開圖片牆） -->
    <DataGridTemplateColumn Header="Image Selection" Width="250">
        <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
                <Button Content="{Binding SelectedImageName}"
                        HorizontalAlignment="Stretch"
                        Padding="10,6"
                        Click="OpenPicker_Click"/>
            </DataTemplate>
        </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 3. SRAM Start Address（唯讀，由工具計算） -->
    <DataGridTemplateColumn Header="SRAM Start Address" Width="160">
        <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding SramStartAddress}" VerticalAlignment="Center"/>
            </DataTemplate>
        </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 4. OSD Selection（1..30；0=未選） -->
    <DataGridTemplateColumn Header="OSD Selection" Width="140">
        <DataGridTemplateColumn.CellTemplate>
            <DataTemplate>
                <ComboBox Width="120"
                          SelectedValuePath="."
                          SelectedValue="{Binding OsdTargetIndex, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
                    <ComboBox.ItemsSource>
                        <Binding Path="DataContext.OsdOptions" RelativeSource="{RelativeSource AncestorType=Window}"/>
                    </ComboBox.ItemsSource>
                </ComboBox>
            </DataTemplate>
        </DataGridTemplateColumn.CellTemplate>
    </DataGridTemplateColumn>

    <!-- 5. OSDx_EN -->
    <DataGridCheckBoxColumn Header="OSDx_EN"
                            Binding="{Binding IsOsdEnabled, Mode=TwoWay}"
                            Width="90"/>

    <!-- 6. HPos -->
    <DataGridTextColumn Header="HPos" Width="90"
                        Binding="{Binding HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>

    <!-- 7. VPos -->
    <DataGridTextColumn Header="VPos" Width="90"
                        Binding="{Binding VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
</DataGrid>