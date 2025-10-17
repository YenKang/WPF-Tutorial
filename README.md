<GroupBox Header="Gray Level Color"
          Visibility="{Binding IsGrayColorVisible, Converter={StaticResource BoolToVis}}">
  <StackPanel Orientation="Horizontal" Margin="12">
    <TextBlock Text="R" VerticalAlignment="Center" Margin="0,0,6,0"/>
    <ComboBox Width="60" IsEditable="False"
              ItemsSource="{Binding GrayColorVM.BoolOptions}"
              SelectedItem="{Binding GrayColorVM.R, Mode=TwoWay}"/>

    <TextBlock Text="G" VerticalAlignment="Center" Margin="12,0,6,0"/>
    <ComboBox Width="60" IsEditable="False"
              ItemsSource="{Binding GrayColorVM.BoolOptions}"
              SelectedItem="{Binding GrayColorVM.G, Mode=TwoWay}"/>

    <TextBlock Text="B" VerticalAlignment="Center" Margin="12,0,6,0"/>
    <ComboBox Width="60" IsEditable="False"
              ItemsSource="{Binding GrayColorVM.BoolOptions}"
              SelectedItem="{Binding GrayColorVM.B, Mode=TwoWay}"/>

    <Button Content="Set" Width="60" Height="30" Margin="12,0,0,0"
            Command="{Binding GrayColorVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>