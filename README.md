<!-- Gray Reverse Control -->
<GroupBox Header="{Binding GrayReverseVM.Title}"
          Visibility="{Binding IsGrayReverseVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,12,0,0">
  <StackPanel Orientation="Horizontal" Margin="12">
    <CheckBox Content="{Binding GrayReverseVM.Title}"
              IsChecked="{Binding GrayReverseVM.Enable, Mode=TwoWay}"
              VerticalAlignment="Center" Margin="0,0,12,0"/>
    <Button Content="Set" Width="60" Height="30"
            Command="{Binding GrayReverseVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>