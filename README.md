<Grid Margin="16">
  <!-- 原本：Auto, *, Auto
       變更：Auto(ICON_DL_SEL), Auto(OSD_EN), *(DataGrid), Auto(Buttons) -->
  <Grid.RowDefinitions>
    <RowDefinition Height="Auto"/>  <!-- Row 0: ICON_DL_SEL Summary（新） -->
    <RowDefinition Height="Auto"/>  <!-- Row 1: OSD_EN Summary（原本 Row 0 往下移） -->
    <RowDefinition Height="*"/>     <!-- Row 2: 主表格 9 欄 -->
    <RowDefinition Height="Auto"/>  <!-- Row 3: OK / Cancel -->
  </Grid.RowDefinitions>

===============
  <!-- ===== Row 0：ICON_DL_SEL Summary（由 Image Selection 推得，30→1） ===== -->
  <StackPanel Grid.Row="0" Margin="0,0,0,8">
    <TextBlock Text="ICON_DL_SEL Summary (30 → 1)"
               FontSize="14" FontWeight="Bold" Foreground="#333"
               Margin="0,0,0,4"/>
    <!-- 右→左顯示 30..1 -->
    <Grid FlowDirection="RightToLeft">
      <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
      </Grid.RowDefinitions>

      <!-- 上排：Index -->
      <ItemsControl Grid.Row="0" ItemsSource="{Binding IconSlots}">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <UniformGrid Columns="30"/>
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Border Padding="3" Margin="1" BorderBrush="#D0D5DA" BorderThickness="1" Background="#F8FAFC" CornerRadius="2">
              <TextBlock Text="{Binding IconIndex}" HorizontalAlignment="Center" FontSize="12" Foreground="#444"/>
            </Border>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>

      <!-- 下排：是否有選圖（IconDlSel/HasImage → 1/0） -->
      <ItemsControl Grid.Row="1" ItemsSource="{Binding IconSlots}">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <UniformGrid Columns="30"/>
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Border Padding="3" Margin="1" BorderBrush="#D0D5DA" BorderThickness="1" Background="White" CornerRadius="2">
              <!-- 如果你尚未改名，就先用 HasImage；之後換回 IconDlSel 也只改這一行 -->
              <TextBlock Text="{Binding IconDlSel, Converter={StaticResource Bool01}}"
                         FontSize="12" FontWeight="SemiBold"
                         HorizontalAlignment="Center" VerticalAlignment="Center" Foreground="#222"/>
            </Border>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </Grid>

    <!-- 細灰分隔線 -->
    <Border Height="1" Margin="0,8,0,0" Background="#E0E4E9" SnapsToDevicePixels="True"/>
  </StackPanel>