<Border ...>  <!-- 你的邊框與樣式同前，省略 -->
  <Grid>
    <!-- 圖片 + 名稱 -->
    <StackPanel>
      <Image Source="{Binding ImagePath}" Height="90" Stretch="UniformToFill"/>
      <TextBlock Text="{Binding Name}" Margin="0,6,0,0" HorizontalAlignment="Center"
                 TextTrimming="CharacterEllipsis"/>
    </StackPanel>

    <!-- 右上角：已被使用角標（仍可選） -->
    <Border Background="#FFE8B5" CornerRadius="3"
            HorizontalAlignment="Right" VerticalAlignment="Top"
            Margin="0,0,4,0" Padding="4,1" Visibility="Collapsed">
      <Border.Visibility>
        <Binding Path="IsUsedByOthers" Converter="{StaticResource BoolToVisibilityConverter}"/>
      </Border.Visibility>
      <TextBlock Text="已被使用" FontSize="10" Foreground="#7A4B00"/>
    </Border>
  </Grid>
</Border>