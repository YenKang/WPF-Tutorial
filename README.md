<GroupBox Header="Background Gray Level"
          Visibility="{Binding IsBgGrayLevelVisible, Converter={utilityConv:BoolToVisibilityConverter}}">
  <StackPanel Orientation="Horizontal">
    <ComboBox ItemsSource="{Binding BgGrayLevelVM.D2Options}" SelectedItem="{Binding BgGrayLevelVM.D2}" />
    <ComboBox ItemsSource="{Binding BgGrayLevelVM.D1Options}" SelectedItem="{Binding BgGrayLevelVM.D1}" />
    <ComboBox ItemsSource="{Binding BgGrayLevelVM.D0Options}" SelectedItem="{Binding BgGrayLevelVM.D0}" />
    <Button Content="Set" Command="{Binding BgGrayLevelVM.ApplyCommand}" />
  </StackPanel>
</GroupBox>