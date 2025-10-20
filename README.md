<!-- P1 -->
<TextBlock Grid.Row="1" Grid.Column="0" Text="P1" VerticalAlignment="Center"/>

<!-- R -->
<ComboBox Grid.Row="1" Grid.Column="1"
          ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
          SelectedItem="{Binding FlickerRGBVM.Planes[0].R, Mode=TwoWay}"
          ItemStringFormat="{}{0:X}"
          Width="70" Margin="1">
  <ComboBox.Style>
    <Style TargetType="ComboBox">
      <Setter Property="BorderBrush" Value="Red"/>
      <Setter Property="BorderThickness" Value="2"/>
      <Setter Property="Foreground" Value="Red"/>
      <Setter Property="FontWeight" Value="Bold"/>
    </Style>
  </ComboBox.Style>
</ComboBox>

<!-- G -->
<ComboBox Grid.Row="1" Grid.Column="2"
          ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
          SelectedItem="{Binding FlickerRGBVM.Planes[0].G, Mode=TwoWay}"
          ItemStringFormat="{}{0:X}"
          Width="70" Margin="1">
  <ComboBox.Style>
    <Style TargetType="ComboBox">
      <Setter Property="BorderBrush" Value="Green"/>
      <Setter Property="BorderThickness" Value="2"/>
      <Setter Property="Foreground" Value="Green"/>
      <Setter Property="FontWeight" Value="Bold"/>
    </Style>
  </ComboBox.Style>
</ComboBox>

<!-- B -->
<ComboBox Grid.Row="1" Grid.Column="3"
          ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
          SelectedItem="{Binding FlickerRGBVM.Planes[0].B, Mode=TwoWay}"
          ItemStringFormat="{}{0:X}"
          Width="70" Margin="1">
  <ComboBox.Style>
    <Style TargetType="ComboBox">
      <Setter Property="BorderBrush" Value="Blue"/>
      <Setter Property="BorderThickness" Value="2"/>
      <Setter Property="Foreground" Value="Blue"/>
      <Setter Property="FontWeight" Value="Bold"/>
    </Style>
  </ComboBox.Style>
</ComboBox>