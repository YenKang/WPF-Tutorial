<GroupBox Header="{Binding LineExclamCursorVM.Title}" Margin="0,12,0,0"
          Visibility="{Binding IsLineExclamCursorVisible,
                               Converter={utilityConv:BoolToVisibilityConverter}}">
  <StackPanel Margin="12" Orientation="Vertical">

    <!-- Exclamation BG -->
    <CheckBox Content="Exclamation BG = White (unchecked = Black)"
              IsChecked="{Binding LineExclamCursorVM.ExclamWhiteBG, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowExclamBg,
                                   Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <!-- First Line Color -->
    <CheckBox Content="First Line Color = White (unchecked = Black)"
              IsChecked="{Binding LineExclamCursorVM.LineSelWhite, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowLineSel,
                                   Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <!-- Cursor / Diagonal -->
    <CheckBox Content="Cursor Configurable Enable"
              IsChecked="{Binding LineExclamCursorVM.CursorEn, Mode=TwoWay}"
              Margin="0,0,0,6"/>
    <CheckBox Content="Diagonal Enable"
              IsChecked="{Binding LineExclamCursorVM.DiagonalEn, Mode=TwoWay}"
              Margin="0,0,0,6"/>

    <!-- ############  新增：Cursor H/V Position  ############ -->

    <!-- H POS -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,6"
                Visibility="{Binding LineExclamCursorVM.IsHPosVisible,
                                     Converter={utilityConv:BoolToVisibilityConverter}}">
      <TextBlock Text="Cursor H Pos:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <TextBox Width="100"
               Text="{Binding LineExclamCursorVM.HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
               ToolTip="{Binding LineExclamCursorVM.HPosRangeText}"/>
    </StackPanel>

    <!-- V POS -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,6"
                Visibility="{Binding LineExclamCursorVM.IsVPosVisible,
                                     Converter={utilityConv:BoolToVisibilityConverter}}">
      <TextBlock Text="Cursor V Pos:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <TextBox Width="100"
               Text="{Binding LineExclamCursorVM.VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
               ToolTip="{Binding LineExclamCursorVM.VPosRangeText}"/>
    </StackPanel>

    <!-- Set -->
    <Button Content="Set" Width="80" Height="30" Margin="0,8,0,0"
            Command="{Binding LineExclamCursorVM.ApplyCommand}"/>

  </StackPanel>
</GroupBox>