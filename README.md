<GroupBox Header="{Binding LineExclamCursorVM.Title}"
          Margin="0,12,0,0"
          Visibility="{Binding IsLineExclamCursorVisible, Converter={utilityConv:BoolToVisibilityConverter}}">

  <StackPanel Margin="12" Orientation="Vertical">

    <!-- Exclamation BG（只有這張圖會顯示） -->
    <CheckBox Content="Exclamation BG = White (unchecked = Black)"
              IsChecked="{Binding LineExclamCursorVM.ExclamWhiteBG, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowExclamBg, Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <!-- 其他三項：這張圖 JSON 沒開，會被隱藏 -->
    <CheckBox Content="Line first = White (unchecked = Black)"
              IsChecked="{Binding LineExclamCursorVM.LineSelWhite, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowLineSel, Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <CheckBox Content="Diagonal Enable (1=On)"
              IsChecked="{Binding LineExclamCursorVM.DiagonalEn, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowDiagonal, Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <CheckBox Content="Cursor Enable (1=On)"
              IsChecked="{Binding LineExclamCursorVM.CursorEn, Mode=TwoWay}"
              Visibility="{Binding LineExclamCursorVM.ShowCursor, Converter={utilityConv:BoolToVisibilityConverter}}"/>

    <Button Content="Set" Width="80" Height="30" Margin="0,8,0,0"
            Command="{Binding LineExclamCursorVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>