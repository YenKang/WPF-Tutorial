<!-- BIST Mode 開關區 -->
<GroupBox Header="BIST Mode" Style="{StaticResource SectionGroupBox}">
  <!-- ↓ 這是 GroupBox 的內容區（真正顯示的地方） ↓ -->
  <StackPanel Orientation="Horizontal" Spacing="8">
    
    <!-- Toggle 開關 -->
    <ToggleButton x:Name="BistModeToggle"
                  Width="90" Height="32"
                  FontWeight="Bold"
                  Content="{Binding IsBistEnabled, Converter={StaticResource BoolToTextConverter}, ConverterParameter='On,Off'}"
                  IsChecked="{Binding IsBistEnabled, Mode=TwoWay}"
                  HorizontalAlignment="Left"/>

    <!-- 普通按鈕 -->
    <Button Content="Enable Mode"
            Padding="12,6"
            Margin="8,0,0,0"
            Command="{Binding EnableBistCommand}"/>

  </StackPanel>
</GroupBox>
