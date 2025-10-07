<Border BorderThickness="2" BorderBrush="Gray" CornerRadius="6" Padding="10" Margin="10">
    <StackPanel>
        <TextBlock Text="BIST_PT_Level [9:0]" FontWeight="Bold" FontSize="14" Margin="0,0,0,8"/>
        <StackPanel Orientation="Vertical" Margin="4,0,0,0">
            <TextBlock Text="Gray Level" Margin="0,0,0,4"/>
            <!-- 預設 0x000~0x3FF、步進 1；Value 綁到 VM -->
            <local:HexNumericUpDown Min="0x000" Max="0x3FF" Step="1"
                                    Value="{Binding GrayLevel, Mode=TwoWay}"/>
            <Button Content="Set" Width="90" Margin="0,10,0,0"
                    Command="{Binding SetCommand}"/>
        </StackPanel>
    </StackPanel>
</Border>