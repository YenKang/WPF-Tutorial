<ComboBox Width="60" ItemsSource="{Binding D2Options}"
          SelectedItem="{Binding FCNT1_D2, Mode=TwoWay}" />
<ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding NibbleOptions}"
          SelectedItem="{Binding FCNT1_D1, Mode=TwoWay}" />
<ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding NibbleOptions}"
          SelectedItem="{Binding FCNT1_D0, Mode=TwoWay}" />
<TextBlock Margin="12,0,0,0" Text="{Binding FCNT1, StringFormat=0x{0:X3}}" />