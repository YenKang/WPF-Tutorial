<StackPanel Orientation="Horizontal" VerticalAlignment="Center" Margin="20">
    <TextBlock Text="Enable BIST Mode" 
               VerticalAlignment="Center"
               Foreground="White"
               FontSize="16"
               Margin="0,0,10,0"/>
    <ToggleSwitch Header="" 
                  OnContent="On" 
                  OffContent="Off"
                  IsOn="{Binding IsBistEnabled}"
                  Width="60"
                  Height="30"/>
</StackPanel>