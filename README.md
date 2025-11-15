<!-- 3. SRAM Start Address -->
<DataGridTemplateColumn Header="SRAM START ADDRESS" Width="160">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding SramStartAddress}"
                       HorizontalAlignment="Center"
                       VerticalAlignment="Center"/>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>

<!-- 3.5 視覺用分隔欄位（不綁任何資料） -->
<DataGridTemplateColumn Width="12"
                        IsReadOnly="True"
                        CanUserSort="False"
                        CanUserReorder="False"
                        CanUserResize="False">
    <!-- Header 部分：畫一個淡灰色的直條 -->
    <DataGridTemplateColumn.HeaderTemplate>
        <DataTemplate>
            <Border Background="#FFE0E0E0"
                    Margin="0,4,0,0"   <!-- 上面跟其他 Header 留一點距離 -->
                    Height="16"/>
        </DataTemplate>
    </DataGridTemplateColumn.HeaderTemplate>

    <!-- 資料列部分：同樣畫一條淡灰色直條，從上到下連成一條帶子 -->
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <Border Background="#FFE0E0E0"
                    Margin="0,0,0,0"/>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>

<!-- 4. OSD #（原本就有的欄位，保持不變） -->
<DataGridTemplateColumn Header="OSD #" Width="120">
    <!-- 你的 CellTemplate... -->
</DataGridTemplateColumn>