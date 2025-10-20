<!-- ▼ Flicker RGB Control GroupBox ▼ -->
<GroupBox Header="Flicker Plane RGB Level"
          Visibility="{Binding IsFlickerRGBVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,12,0,0">
  <Grid Margin="8">
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
      <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
      <ColumnDefinition Width="40"/>   <!-- 標籤 (P1..P4) -->
      <ColumnDefinition Width="80"/>   <!-- R -->
      <ColumnDefinition Width="80"/>   <!-- G -->
      <ColumnDefinition Width="80"/>   <!-- B -->
      <ColumnDefinition Width="*"/>    <!-- Set 按鈕 -->
    </Grid.ColumnDefinitions>

    <!-- Row 0: 標題列 -->
    <TextBlock Grid.Row="0" Grid.Column="0" />
    <TextBlock Grid.Row="0" Grid.Column="1" Text="R" FontWeight="Bold" HorizontalAlignment="Center"/>
    <TextBlock Grid.Row="0" Grid.Column="2" Text="G" FontWeight="Bold" HorizontalAlignment="Center"/>
    <TextBlock Grid.Row="0" Grid.Column="3" Text="B" FontWeight="Bold" HorizontalAlignment="Center"/>

    <!-- Row 1: P1 -->
    <TextBlock Grid.Row="1" Grid.Column="0" Text="P1" VerticalAlignment="Center"/>
    <ComboBox Grid.Row="1" Grid.Column="1"
              ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[0].R, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="1" Grid.Column="2"
              ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[0].G, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="1" Grid.Column="3"
              ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[0].B, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>

    <!-- Row 2: P2 -->
    <TextBlock Grid.Row="2" Grid.Column="0" Text="P2" VerticalAlignment="Center"/>
    <ComboBox Grid.Row="2" Grid.Column="1"
              ItemsSource="{Binding FlickerRGBVM.Planes[1].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[1].R, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="2" Grid.Column="2"
              ItemsSource="{Binding FlickerRGBVM.Planes[1].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[1].G, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="2" Grid.Column="3"
              ItemsSource="{Binding FlickerRGBVM.Planes[1].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[1].B, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>

    <!-- Row 3: P3 -->
    <TextBlock Grid.Row="3" Grid.Column="0" Text="P3" VerticalAlignment="Center"/>
    <ComboBox Grid.Row="3" Grid.Column="1"
              ItemsSource="{Binding FlickerRGBVM.Planes[2].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[2].R, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="3" Grid.Column="2"
              ItemsSource="{Binding FlickerRGBVM.Planes[2].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[2].G, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="3" Grid.Column="3"
              ItemsSource="{Binding FlickerRGBVM.Planes[2].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[2].B, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>

    <!-- Row 4: P4 -->
    <TextBlock Grid.Row="4" Grid.Column="0" Text="P4" VerticalAlignment="Center"/>
    <ComboBox Grid.Row="4" Grid.Column="1"
              ItemsSource="{Binding FlickerRGBVM.Planes[3].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[3].R, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="4" Grid.Column="2"
              ItemsSource="{Binding FlickerRGBVM.Planes[3].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[3].G, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>
    <ComboBox Grid.Row="4" Grid.Column="3"
              ItemsSource="{Binding FlickerRGBVM.Planes[3].LevelOptions}"
              SelectedItem="{Binding FlickerRGBVM.Planes[3].B, Mode=TwoWay}"
              ItemStringFormat="{}{0:X}" Width="70"/>

    <!-- Set 按鈕 -->
    <Button Grid.Row="1" Grid.RowSpan="4" Grid.Column="4"
            Content="Set" Width="80" Height="30"
            HorizontalAlignment="Left" VerticalAlignment="Center"
            Command="{Binding FlickerRGBVM.ApplyCommand}"/>
  </Grid>
</GroupBox>