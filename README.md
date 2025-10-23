<GroupBox Header="B/W Loop (FCNT2)"
          Margin="0,8,0,0"
          Visibility="{Binding IsBWLoopVisible, Converter={StaticResource BoolToVisibility}}">
  <StackPanel Margin="10">
    <StackPanel Orientation="Horizontal">
      <TextBlock Text="FCNT2:" Width="80" VerticalAlignment="Center"/>
      <ComboBox Width="60" Margin="0,0,6,0"
                ItemsSource="{Binding BWLoopVM.D2Options}"
                SelectedItem="{Binding BWLoopVM.FCNT2_D2, Mode=TwoWay}" />
      <ComboBox Width="60" Margin="0,0,6,0"
                ItemsSource="{Binding BWLoopVM.D1Options}"
                SelectedItem="{Binding BWLoopVM.FCNT2_D1, Mode=TwoWay}" />
      <ComboBox Width="60" Margin="0,0,12,0"
                ItemsSource="{Binding BWLoopVM.D0Options}"
                SelectedItem="{Binding BWLoopVM.FCNT2_D0, Mode=TwoWay}" />
      <TextBlock VerticalAlignment="Center"
                 Text="{Binding BWLoopVM.FCNT2_Value, StringFormat=0x{0:X3}}"
                 FontWeight="SemiBold" Margin="4,0,0,0"/>
      <TextBlock VerticalAlignment="Center"
                 Text="{Binding BWLoopVM.FCNT2_Value}" Foreground="#777" Margin="6,0,0,0"/>
    </StackPanel>

    <Button Content="Set FCNT2" Width="150" Margin="0,10,0,0"
            Command="{Binding BWLoopVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>