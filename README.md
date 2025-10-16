<!-- D2 -->
<ComboBox IsEditable="False"
          ItemsSource="{Binding GrayLevelVM.D2Options}"
          SelectedItem="{Binding GrayLevelVM.D2, Mode=TwoWay, TargetNullValue={x:Static sys:Int32:0}, ValidatesOnExceptions=False}"
          ItemStringFormat="{}{0:X}" Width="50" Margin="4,0"/>

<!-- D1 -->
<ComboBox IsEditable="False"
          ItemsSource="{Binding GrayLevelVM.D1Options}"
          SelectedItem="{Binding GrayLevelVM.D1, Mode=TwoWay, TargetNullValue={x:Static sys:Int32:0}, ValidatesOnExceptions=False}"
          ItemStringFormat="{}{0:X}" Width="50" Margin="4,0"/>

<!-- D0 -->
<ComboBox IsEditable="False"
          ItemsSource="{Binding GrayLevelVM.D0Options}"
          SelectedItem="{Binding GrayLevelVM.D0, Mode=TwoWay, TargetNullValue={x:Static sys:Int32:0}, ValidatesOnExceptions=False}"
          ItemStringFormat="{}{0:X}" Width="50" Margin="4,0"/>