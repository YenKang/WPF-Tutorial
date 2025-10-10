<StackPanel Margin="12">
    <!-- (1) 標題在 Border 上方 -->
    <TextBlock Text="BIST_PT_Level[9:0]"
               FontWeight="Bold" FontSize="16" Margin="0,0,0,8"/>

    <Border Background="#FFFFFF" CornerRadius="8" Padding="12"
            BorderBrush="#E0E3E7" BorderThickness="1">

        <Grid RowDefinitions="Auto,Auto,Auto,Auto" ColumnDefinitions="*">

            <!-- (2) 第一行顯示：挑選 pattern -->
            <TextBlock Grid.Row="0" FontWeight="SemiBold" Margin="0,0,0,8">
                <TextBlock.Text>
                    <MultiBinding StringFormat="挑選 pattern：P{0} — {1}">
                        <Binding Path="SelectedPattern.Index" TargetNullValue="-" />
                        <Binding Path="SelectedPattern.Name"  TargetNullValue="(none)" />
                    </MultiBinding>
                </TextBlock.Text>
            </TextBlock>

            <!-- (3) 三個十六進位 Digit，移除 D[9:8]/D[7:4]/D[3:0] 標籤 -->
            <StackPanel Grid.Row="1" Orientation="Horizontal" Spacing="10">
                <!-- D[9:8]：0~3 -->
                <ComboBox Width="60"
                          ItemsSource="{Binding High2BitList}"
                          SelectedItem="{Binding Digit2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
                <!-- D[7:4]：0~F -->
                <ComboBox Width="60"
                          ItemsSource="{Binding LowNibbleList}"
                          SelectedItem="{Binding Digit1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
                <!-- D[3:0]：0~F -->
                <ComboBox Width="60"
                          ItemsSource="{Binding LowNibbleList}"
                          SelectedItem="{Binding Digit0, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
            </StackPanel>

            <!-- 顯示目前十六進位：例如 3 FF -->
            <TextBlock Grid.Row="2" Margin="0,10,0,0"
                       FontFamily="Consolas" FontSize="20"
                       Text="{Binding HexDisplay}" />

            <!-- (4) 在 3 FF 下方，顯示 GrayLevel = 十進位 -->
            <TextBlock Grid.Row="3" Margin="0,4,0,0"
                       Text="{Binding GrayLevelDisplay}" />
        </Grid>
    </Border>
</StackPanel>
