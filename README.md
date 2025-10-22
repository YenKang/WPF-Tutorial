<UserControl x:Class="BistMode.Views.ChessBoardControl"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <GroupBox Header="Chessboard" Padding="10" Margin="0,8,0,0">
    <StackPanel>

      <!-- Resolution -->
      <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
        <TextBlock Text="Resolution:" Width="100"/>
        <TextBlock Text="{Binding HRes}" Width="60"/>
        <TextBlock Text="×" Margin="6,0"/>
        <TextBlock Text="{Binding VRes}" Width="60"/>
      </StackPanel>

      <!-- M / N -->
      <StackPanel Orientation="Horizontal" Margin="0,0,0,10">
        <TextBlock Text="M (H split):" Width="100"/>
        <TextBox Width="80" Text="{Binding M, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>

        <TextBlock Text="N (V split):" Width="100" Margin="24,0,0,0"/>
        <TextBox Width="80" Text="{Binding N, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
      </StackPanel>

      <!-- Set -->
      <Button Content="Set"
              Width="120"
              Command="{Binding ApplyCommand}"/>
    </StackPanel>
  </GroupBox>
</UserControl>