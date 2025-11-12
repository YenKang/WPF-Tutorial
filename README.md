<!-- 🟦 Row 0：Icon_DL_SEL Summary（由 Image Selection 推得，30→1） -->
<StackPanel Grid.Row="0" Margin="0,0,0,8">
  <TextBlock Text="Icon_DL_SEL Summary (30 → 1)"
             FontSize="14"
             FontWeight="Bold"
             Foreground="#333"
             Margin="0,0,0,4"/>

  <Grid FlowDirection="RightToLeft">
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>

    <!-- 上排：Icon Index (30→1) -->
    <ItemsControl Grid.Row="0" ItemsSource="{Binding IconSlots}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <UniformGrid Columns="30"/>
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Border Padding="3" Margin="1"
                  BorderBrush="#D0D5DA" BorderThickness="1"
                  Background="#F8FAFC" CornerRadius="2">
            <TextBlock Text="{Binding IconIndex}"
                       HorizontalAlignment="Center"
                       FontSize="12"
                       Foreground="#444"/>
          </Border>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- 下排：是否有選圖（Icon_DL_SEL → 1 / 0） -->
    <ItemsControl Grid.Row="1" ItemsSource="{Binding IconSlots}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <UniformGrid Columns="30"/>
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Border Padding="3" Margin="1"
                  BorderBrush="#D0D5DA" BorderThickness="1"
                  Background="White" CornerRadius="2">
            <TextBlock Text="{Binding IconDlSel, Converter={StaticResource Bool01}}"
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