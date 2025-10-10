<StackPanel Margin="12">
  <!-- 標題獨立在 Border 上方 -->
  <TextBlock Text="BIST_PT_Level[9:0]"
             FontWeight="Bold" FontSize="16" Margin="0,0,0,8"/>

  <!-- Border 只放一個孩子 -->
  <Border Background="#FFFFFF" CornerRadius="8" Padding="12"
          BorderBrush="#E0E3E7" BorderThickness="1">
    <Grid>
      <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
      </Grid.RowDefinitions>

      <!-- 第一行：挑選 pattern -->
      <TextBlock Grid.Row="0" FontWeight="SemiBold" Margin="0,0,0,8">
        <TextBlock.Text>
          <MultiBinding StringFormat="挑選 pattern：P{0} — {1}">
            <Binding Path="SelectedPattern.Index" TargetNullValue="-" />
            <Binding Path="SelectedPattern.Name"  TargetNullValue="(none)" />
          </MultiBinding>
        </TextBlock.Text>
      </TextBlock>

      <!-- 三個十六進位 Digit（依你的截圖綁法） -->
      <StackPanel Grid.Row="1" Orientation="Horizontal" Margin="0,0,0,6">
        <!-- D[9:8] -->
        <ComboBox Width="60"
                  ItemsSource="{Binding LowDigitList}"
                  SelectedItem="{Binding D98, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                  Margin="0,0,10,0"/>
        <!-- D[7:4] -->
        <ComboBox Width="60"
                  ItemsSource="{Binding LowDigitList}"
                  SelectedItem="{Binding D74, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                  Margin="0,0,10,0"/>
        <!-- D[3:0] -->
        <ComboBox Width="60"
                  ItemsSource="{Binding LowDigitList}"
                  SelectedItem="{Binding D30, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
      </StackPanel>

      <!-- 顯示 3 FF -->
      <TextBlock Grid.Row="2" Margin="0,8,0,0"
                 FontFamily="Consolas" FontSize="20"
                 Text="{Binding HexDisplay}" />

      <!-- 顯示 GrayLevel = 十進位（例如 1023） -->
      <TextBlock Grid.Row="3" Margin="0,4,0,0"
                 Text="{Binding GrayLevelDisplay}" />
    </Grid>
  </Border>
</StackPanel>
