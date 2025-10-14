private void OnDebugJsonClick(object sender, RoutedEventArgs e)
{
    var vm = DataContext as BistModeViewModel;
    if (vm == null)
    {
        MessageBox.Show("DataContext = null 或不是 BistModeViewModel");
        return;
    }
    vm.DebugJson();   // <- 直接叫你的方法
}

＝＝＝＝＝＝

<Button Content="Debug JSON & Map"
        Width="200" Height="32" Margin="8"
        Click="OnDebugJsonClick"/>
