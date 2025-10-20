<GroupBox Header="Cursor / Diagonal"
          Visibility="{Binding IsLineExclamCursorVisible,
                               Converter={StaticResource BoolToVisibilityConverter}}">
  <StackPanel Margin="6">

    <!-- 1️⃣ 開關 -->
    <CheckBox Content="Cursor Configurable Enable"
              IsChecked="{Binding LineExclamCursorVM.CursorEn, Mode=TwoWay}"
              Margin="0,0,0,6"/>
    <CheckBox Content="Diagonal Enable"
              IsChecked="{Binding LineExclamCursorVM.DiagonalEn, Mode=TwoWay}"
              Margin="0,0,0,6"/>

    <!-- 2️⃣ Cursor Position: Horizontal -->
    <StackPanel Orientation="Horizontal"
                Visibility="{Binding LineExclamCursorVM.IsHPosVisible,
                                     Converter={StaticResource BoolToVisibilityConverter}}">
      <TextBlock Text="Cursor H Pos:" Margin="0,0,6,0" VerticalAlignment="Center"/>
      <TextBox Width="100"
               Text="{Binding LineExclamCursorVM.HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- 3️⃣ Cursor Position: Vertical -->
    <StackPanel Orientation="Horizontal"
                Visibility="{Binding LineExclamCursorVM.IsVPosVisible,
                                     Converter={StaticResource BoolToVisibilityConverter}}">
      <TextBlock Text="Cursor V Pos:" Margin="0,0,6,0" VerticalAlignment="Center"/>
      <TextBox Width="100"
               Text="{Binding LineExclamCursorVM.VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- 4️⃣ Set 按鈕 -->
    <Button Content="Set" Width="80" Height="28" Margin="0,10,0,0"
            Command="{Binding LineExclamCursorVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>