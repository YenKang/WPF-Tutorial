<!-- 圖片牆 -->
<ItemsControl x:Name="ic">
  <!-- 以 WrapPanel 平鋪 -->
  <ItemsControl.ItemsPanel>
    <ItemsPanelTemplate>
      <WrapPanel />
    </ItemsPanelTemplate>
  </ItemsControl.ItemsPanel>

  <!-- 單一圖片卡片樣式 -->
  <ItemsControl.ItemTemplate>
    <DataTemplate>
      <Grid>
        <!-- 外框：本列目前選中（藍色外框） -->
        <Border Margin="6"
                CornerRadius="4"
                BorderThickness="2">
          <Border.Style>
            <Style TargetType="Border">
              <Setter Property="BorderBrush" Value="Transparent"/>
              <Style.Triggers>
                <!-- 本列目前選的圖（藍框） -->
                <DataTrigger Binding="{Binding IsCurrentSelected}" Value="True">
                  <Setter Property="BorderBrush" Value="#2F80ED"/>
                </DataTrigger>
              </Style.Triggers>
            </Style>
          </Border.Style>

          <StackPanel Width="140" Height="140">
            <Image Source="{Binding ImagePath}"
                   Stretch="UniformToFill"
                   Height="100"/>
            <TextBlock Text="{Binding Name}"
                       HorizontalAlignment="Center"
                       Margin="0,6,0,0"
                       TextTrimming="CharacterEllipsis"/>
          </StackPanel>
        </Border>

        <!-- 右上角標籤：其他列使用中（僅提示，不禁用） -->
        <Border HorizontalAlignment="Right"
                VerticalAlignment="Top"
                Margin="0,8,8,0"
                Padding="4,2"
                Background="#FFF5B5"
                CornerRadius="3">
          <Border.Style>
            <Style TargetType="Border">
              <Setter Property="Visibility" Value="Collapsed"/>
              <Style.Triggers>
                <DataTrigger Binding="{Binding IsUsedByOthers}" Value="True">
                  <Setter Property="Visibility" Value="Visible"/>
                </DataTrigger>
              </Style.Triggers>
            </Style>
          </Border.Style>
          <TextBlock Text="其他列使用中"
                     FontSize="10"
                     Foreground="#7A4B00"
                     FontWeight="Bold"/>
        </Border>
      </Grid>
    </DataTemplate>
  </ItemsControl.ItemTemplate>

  <!-- 點擊事件（允許重複選） -->
  <ItemsControl.ItemContainerStyle>
    <Style TargetType="ContentPresenter">
      <EventSetter Event="MouseLeftButtonUp" Handler="OnPick"/>
    </Style>
  </ItemsControl.ItemContainerStyle>
</ItemsControl>