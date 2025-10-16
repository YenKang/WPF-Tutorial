、<!-- 🧱 Gray Level Control -->
<GroupBox Header="Gray Level Control"
          Visibility="{Binding IsGrayLevelVisible, Converter={utilityConv:BoolToVisibilityConverter}}">
    <StackPanel Orientation="Horizontal" Margin="12">

        <TextBlock Text="GrayLevel [9:0]"
                   FontWeight="Bold"
                   VerticalAlignment="Center"
                   Margin="0,0,12,0"/>

        <!-- D2 -->
        <ComboBox Width="50"
                  ItemsSource="{Binding GrayLevelVM.D2Options}"
                  SelectedItem="{Binding GrayLevelVM.D2, Mode=TwoWay}"
                  Margin="4,0"
                  ItemStringFormat="{}{0:X}"/>

        <!-- D1 -->
        <ComboBox Width="50"
                  ItemsSource="{Binding GrayLevelVM.D1Options}"
                  SelectedItem="{Binding GrayLevelVM.D1, Mode=TwoWay}"
                  Margin="4,0"
                  ItemStringFormat="{}{0:X}"/>

        <!-- D0 -->
        <ComboBox Width="50"
                  ItemsSource="{Binding GrayLevelVM.D0Options}"
                  SelectedItem="{Binding GrayLevelVM.D0, Mode=TwoWay}"
                  Margin="4,0"
                  ItemStringFormat="{}{0:X}"/>

        <Button Content="Set"
                Width="60" Height="30"
                Margin="12,0,0,0"
                Command="{Binding GrayLevelVM.ApplyCommand}"/>
    </StackPanel>
</GroupBox>