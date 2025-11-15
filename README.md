<!-- 3. SRAM START ADDRESS（左群組最後一欄，右邊加粗線當分隔） -->
<DataGridTemplateColumn Header="SRAM START ADDRESS" Width="160">

    <!-- Header：右邊畫一條比較明顯的分隔線 -->
    <DataGridTemplateColumn.HeaderStyle>
        <Style TargetType="DataGridColumnHeader">
            <!-- 文字置中，可依你原本 Style 調整 -->
            <Setter Property="HorizontalContentAlignment" Value="Center"/>
            <Setter Property="VerticalContentAlignment"   Value="Center"/>

            <!-- 線的顏色／粗細：這裡是右邊 2px 深灰色 -->
            <Setter Property="BorderBrush"     Value="#FFB0B0B0"/>
            <Setter Property="BorderThickness" Value="0,0,2,0"/>

            <!-- 一點點 padding 讓字不貼邊 -->
            <Setter Property="Padding" Value="4,0,4,0"/>
        </Style>
    </DataGridTemplateColumn.HeaderStyle>

    <!-- Cell：每一列的 SRAM 欄位右邊也畫同樣的線，和 Header 對齊 -->
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
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