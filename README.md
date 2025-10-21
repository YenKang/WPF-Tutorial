<!-- Frame Count (FCNT1 / FCNT2) -->
<StackPanel Margin="0,12,0,0">
  <TextBlock Text="Frame Count" FontWeight="Bold" Margin="0,0,0,6"/>

  <!-- FCNT1 -->
  <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
    <TextBlock Text="FCNT1:" Width="60" VerticalAlignment="Center"/>
    <ComboBox Width="60" ItemsSource="{Binding AutoRunVM.D2Options}"
              SelectedItem="{Binding AutoRunVM.FCNT1_D2, Mode=TwoWay}" />
    <ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding AutoRunVM.NibbleOptions}"
              SelectedItem="{Binding AutoRunVM.FCNT1_D1, Mode=TwoWay}" />
    <ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding AutoRunVM.NibbleOptions}"
              SelectedItem="{Binding AutoRunVM.FCNT1_D0, Mode=TwoWay}" />
    <TextBlock Margin="12,0,0,0" VerticalAlignment="Center"
               Text="{Binding AutoRunVM.FCNT1_D2, StringFormat='0x{0:X}'}"/>
  </StackPanel>

  <!-- FCNT2 -->
  <StackPanel Orientation="Horizontal">
    <TextBlock Text="FCNT2:" Width="60" VerticalAlignment="Center"/>
    <ComboBox Width="60" ItemsSource="{Binding AutoRunVM.D2Options}"
              SelectedItem="{Binding AutoRunVM.FCNT2_D2, Mode=TwoWay}" />
    <ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding AutoRunVM.NibbleOptions}"
              SelectedItem="{Binding AutoRunVM.FCNT2_D1, Mode=TwoWay}" />
    <ComboBox Width="60" Margin="6,0,0,0" ItemsSource="{Binding AutoRunVM.NibbleOptions}"
              SelectedItem="{Binding AutoRunVM.FCNT2_D0, Mode=TwoWay}" />
    <TextBlock Margin="12,0,0,0" VerticalAlignment="Center"
               Text="{Binding AutoRunVM.FCNT2_D2, StringFormat='0x{0:X}'}"/>
  </StackPanel>
</StackPanel>