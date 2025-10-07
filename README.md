<Border BorderThickness="2" BorderBrush="Gray" CornerRadius="5" Padding="10" Margin="20">
    <StackPanel>
        <!-- 標題 -->
        <TextBlock Text="BIST_PT_Level [9:0]"
                   FontWeight="Bold"
                   FontSize="14"
                   Margin="0,0,0,10"/>

        <!-- Gray level 區塊 -->
        <StackPanel Orientation="Vertical" Margin="5">
            <TextBlock Text="Gray Level" FontSize="12" Margin="0,0,0,5"/>

            <StackPanel Orientation="Horizontal" HorizontalAlignment="Left">
                <!-- TextBox 顯示 0x3FF -->
                <TextBox Width="80"
                         Text="{Binding GrayLevelHex, UpdateSourceTrigger=PropertyChanged}"
                         HorizontalContentAlignment="Center"
                         VerticalContentAlignment="Center"
                         FontFamily="Consolas"
                         FontSize="14"
                         MaxLength="5"
                         Margin="0,0,5,0"/>

                <!-- Set 按鈕 -->
                <Button Content="Set"
                        Width="60"
                        Command="{Binding SetCommand}"/>
            </StackPanel>
        </StackPanel>
    </StackPanel>
</Border>