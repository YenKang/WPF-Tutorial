<GroupBox Header="Chessboard" Margin="0,10,0,0">
  <StackPanel Margin="10">

    <!-- Resolution -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Resolution:" Width="100"/>
      <TextBlock Text="{Binding ChessBoardVM.HRes}" Width="60"/>
      <TextBlock Text="×" Margin="6,0"/>
      <TextBlock Text="{Binding ChessBoardVM.VRes}" Width="60"/>
    </StackPanel>

    <!-- M / N (ComboBox) -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,10">
      <TextBlock Text="M (H split):" Width="100"/>
      <ComboBox Width="80"
                ItemsSource="{Binding ChessBoardVM.MOptions}"
                SelectedItem="{Binding ChessBoardVM.M, Mode=TwoWay}" />

      <TextBlock Text="N (V split):" Width="100" Margin="24,0,0,0"/>
      <ComboBox Width="80"
                ItemsSource="{Binding ChessBoardVM.NOptions}"
                SelectedItem="{Binding ChessBoardVM.N, Mode=TwoWay}" />
    </StackPanel>

    <Button Content="Set"
            Width="120"
            Command="{Binding ChessBoardVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>