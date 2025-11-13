<!-- 4. OSD ICON SELECTION（第二階段選圖，從 SRAM 候選圖中挑） -->
<DataGridTemplateColumn Header="OSD ICON SELECTION" Width="160">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <!-- 類似 Image Selection，那種 Button + 文字的做法 -->
            <Button Content="{Binding OsdSelectedImageName}"
                    Padding="4,0"
                    HorizontalAlignment="Stretch"
                    Click="OpenOsdIconPicker_Click"/>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>