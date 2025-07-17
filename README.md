// MusicPageViewModel.cs
public class MusicPageViewModel : INotifyPropertyChanged
{
    private readonly NovaCIDViewModel _mainVM;

    public MusicPageViewModel(NovaCIDViewModel mainViewModel)
    {
        _mainVM = mainViewModel;
    }

    public ICommand ShowLeftKnobInnerCommand => new RelayCommand(ShowLeftKnobInnerImage);

    private void ShowLeftKnobInnerImage()
    {
        _mainVM.LeftKnobInnerImage = "/Assets/music_switch_icon.png";
        _mainVM.IsLeftKnobInnerVisible = true;
    }

    // ... 實作 INotifyPropertyChanged
}


<Button Content="顯示切歌圖示"
        Command="{Binding ShowLeftKnobInnerCommand}" />


<Image Source="{Binding LeftKnobInnerImage}"
       Width="220"
       Height="220"
       Visibility="{Binding IsLeftKnobInnerVisible, Converter={StaticResource BoolToVis}}"
       HorizontalAlignment="Center"
       VerticalAlignment="Center"/>