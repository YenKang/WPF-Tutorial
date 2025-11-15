<!-- 3. SRAM Start Address (工具計算的欄位) -->
<DataGridTemplateColumn Header="SRAM START ADDRESS" Width="160">
    
    <!-- ① Header：在右邊畫一條比較粗的直線，當作「分隔線」 -->
    <DataGridTemplateColumn.HeaderStyle>
        <Style TargetType="DataGridColumnHeader">
            <!-- 保持原本的樣式，如果你已有共用 Style 可以再調整 -->
            <Setter Property="HorizontalContentAlignment" Value="Center"/>
            <Setter Property="BorderBrush" Value="#FFB0B0B0"/>   <!-- 線的顏色，可調淡一點 -->
            <Setter Property="BorderThickness" Value="0,0,2,0"/> <!-- 右邊 Thickness=2 畫粗線 -->
            <Setter Property="Padding" Value="4,0,4,0"/>
        </Style>
    </DataGridTemplateColumn.HeaderStyle>

    <!-- ② CellTemplate：資料列也一樣在右邊畫線，和 Header 對齊 -->
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <!-- 用 Border 包住 TextBlock，在右邊畫出分隔線 -->
            <Border BorderBrush="#FFB0B0B0"
                    BorderThickness="0,0,2,0"
                    Padding="4,0,4,0">
                <TextBlock Text="{Binding SramStartAddress}"
                           HorizontalAlignment="Center"
                           VerticalAlignment="Center"/>
            </Border>
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>

</DataGridTemplateColumn>