<!-- ✅ 第二列：OSD_EN Summary 顏色標註版 -->
<ItemsControl Grid.Row="1" ItemsSource="{Binding Source={StaticResource IconSlotsDesc}}">
  <ItemsControl.ItemsPanel>
    <ItemsPanelTemplate>
      <UniformGrid Columns="30"/>
    </ItemsPanelTemplate>
  </ItemsControl.ItemsPanel>

  <ItemsControl.ItemTemplate>
    <DataTemplate>
      <!-- 每個小格 -->
      <Border x:Name="cell"
              Padding="4" Margin="1"
              CornerRadius="3"
              BorderThickness="1"
              BorderBrush="#D9DEE8"
              Background="#F7F9FC">
        <TextBlock Text="{Binding IsOsdEnabled, Converter={StaticResource Bool01}}"
                   FontSize="13"
                   FontWeight="SemiBold"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"/>
      </Border>

      <!-- 🎨 顏色變化 -->
      <DataTemplate.Triggers>
        <!-- 有勾 (IsOsdEnabled=True) -->
        <DataTrigger Binding="{Binding IsOsdEnabled}" Value="True">
          <Setter TargetName="cell" Property="Background" Value="#2E7D32"/> <!-- 深綠 -->
          <Setter TargetName="cell" Property="BorderBrush" Value="#2E7D32"/>
          <Setter TargetName="cell" Property="TextBlock.Foreground" Value="White"/>
        </DataTrigger>

        <!-- 沒勾 (IsOsdEnabled=False) -->
        <DataTrigger Binding="{Binding IsOsdEnabled}" Value="False">
          <Setter TargetName="cell" Property="Background" Value="#ECEFF1"/> <!-- 淺灰 -->
          <Setter TargetName="cell" Property="BorderBrush" Value="#D0D5DA"/>
          <Setter TargetName="cell" Property="TextBlock.Foreground" Value="#5A6472"/>
        </DataTrigger>
      </DataTemplate.Triggers>
    </DataTemplate>
  </ItemsControl.ItemTemplate>
</ItemsControl>