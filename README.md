<Grid>
    <!-- 其他內容... -->

    <!-- 右上角按鈕 -->
    <Grid HorizontalAlignment="Right" VerticalAlignment="Top" Margin="10">
        <Button Content="⛶" Width="40" Height="30" Click="FullscreenButton_Click"/>
    </Grid>

</Grid>

private void ExitFullscreenButton_Click(object sender, RoutedEventArgs e)
{
    Window win = Window.GetWindow(this);
    if (win != null)
    {
        win.WindowStyle = WindowStyle.SingleBorderWindow;
        win.WindowState = WindowState.Normal;
        win.Topmost = false;
    }
}