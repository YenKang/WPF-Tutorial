<!-- ===== OSD_EN Summary（顯示 30→1、純數字） ===== -->
<StackPanel Grid.Row="0" Margin="0,0,0,8">
  <TextBlock Text="OSD_EN Summary (30 → 1)"
             FontSize="14" FontWeight="Bold" Foreground="#333"
             Margin="0,0,0,4"/>

  <Grid>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>

    <!-- 第一排：顯示 IconIndex（1~30） -->
    <ItemsControl Grid.Row="0" ItemsSource="{Binding IconSlots}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <UniformGrid Columns="30"/>
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Border Padding="3" Margin="1" BorderBrush="#D0D5DA" BorderThickness="1" Background="#F8FAFC" CornerRadius="2">
            <TextBlock Text="{Binding IconIndex}"
                       HorizontalAlignment="Center"
                       FontSize="12"
                       Foreground="#444"/>
          </Border>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- 第二排：顯示 OSD_EN 值（0/1） -->
    <ItemsControl Grid.Row="1" ItemsSource="{Binding IconSlots}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <UniformGrid Columns="30"/>
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Border Padding="3" Margin="1" BorderBrush="#D0D5DA" BorderThickness="1" Background="#FFFFFF" CornerRadius="2">
            <TextBlock Text="{Binding IsOsdEnabled, Converter={StaticResource Bool01}}"
                       FontSize="12"
                       FontWeight="SemiBold"
                       HorizontalAlignment="Center"
                       VerticalAlignment="Center"
                       Foreground="#222"/>
          </Border>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </Grid>
</StackPanel>