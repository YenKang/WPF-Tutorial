<Grid>
    <!-- 其他內容 ... -->

    <Button x:Name="ExitFullScreenButton"
            Content="⤢"
            Width="50" Height="50"
            HorizontalAlignment="Right"
            VerticalAlignment="Top"
            Margin="10"
            FontSize="24"
            Background="Transparent"
            Foreground="White"
            BorderThickness="0"
            Click="ExitFullScreenButton_Click" />
</Grid>


private void ExitFullScreenButton_Click(object sender, RoutedEventArgs e)
{
    var win = Window.GetWindow(this);
    if (win != null)
    {
        win.WindowStyle = WindowStyle.SingleBorderWindow;
        win.WindowState = WindowState.Normal;
    }
}