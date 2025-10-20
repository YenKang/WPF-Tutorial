<!-- P1 -->
<TextBlock Grid.Row="1" Grid.Column="0" Text="P1" VerticalAlignment="Center"/>

<!-- R -->
<Border Grid.Row="1" Grid.Column="1" BorderBrush="Red" BorderThickness="0 0 0 2">
  <ComboBox ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
            SelectedItem="{Binding FlickerRGBVM.Planes[0].R, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Width="70"/>
</Border>

<!-- G -->
<Border Grid.Row="1" Grid.Column="2" BorderBrush="Green" BorderThickness="0 0 0 2">
  <ComboBox ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
            SelectedItem="{Binding FlickerRGBVM.Planes[0].G, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Width="70"/>
</Border>

<!-- B -->
<Border Grid.Row="1" Grid.Column="3" BorderBrush="Blue" BorderThickness="0 0 0 2">
  <ComboBox ItemsSource="{Binding FlickerRGBVM.Planes[0].LevelOptions}"
            SelectedItem="{Binding FlickerRGBVM.Planes[0].B, Mode=TwoWay}"
            ItemStringFormat="{}{0:X}" Width="70"/>
</Border>