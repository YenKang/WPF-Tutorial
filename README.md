<Grid Background="#F7F8FA" UseLayoutRounding="True" SnapsToDevicePixels="True">
  <Grid.ColumnDefinitions>
    <ColumnDefinition Width="380"/>   <!-- 左側固定欄 -->
    <ColumnDefinition Width="12"/>    <!-- 中間縫隙 -->
    <ColumnDefinition Width="*"/>     <!-- 右側舞台 -->
  </Grid.ColumnDefinitions>

  <!-- 左側控制欄 -->
  <ScrollViewer Grid.Column="0" VerticalScrollBarVisibility="Auto">
    <StackPanel Margin="12" Orientation="Vertical" >

      <!-- 1) Toggle 區 -->
      <GroupBox Header="BIST Mode" Margin="0,0,0,12">
        <StackPanel Orientation="Horizontal" Spacing="8">
          <ToggleButton Content="Off" Width="80" Height="28"/>
          <Button Content="Enable Mode" Padding="12,6" Margin="8,0,0,0"/>
        </StackPanel>
      </GroupBox>

      <!-- 2) Pattern 區 -->
      <GroupBox Header="Pattern En" Margin="0,0,0,12">
        <!-- 放你現有 Pattern 選圖內容 -->
        <StackPanel>
          <TextBlock Text="Pattern selection area" Foreground="Gray"/>
        </StackPanel>
      </GroupBox>

      <!-- 3) GrayLevel 區 -->
      <GroupBox Header="BIST_PT_Level [9:0]" Margin="0,0,0,12">
        <!-- 放你 GrayLevel 控制內容 -->
        <StackPanel>
          <TextBlock Text="Gray Level controls here" Foreground="Gray"/>
        </StackPanel>
      </GroupBox>

    </StackPanel>
  </ScrollViewer>

  <!-- 右側固定寬圖片牆 -->
  <Border Grid.Column="2" Padding="12" HorizontalAlignment="Center">
    <Grid Width="960">
      <!-- 放你的圖片牆 XAML -->
      <TextBlock Text="Image Wall Area"
                 Foreground="DarkGray"
                 FontSize="16"
                 HorizontalAlignment="Center"
                 VerticalAlignment="Center"/>
    </Grid>
  </Border>
</Grid>
