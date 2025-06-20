# WPF-Tutorial

<StackPanel VerticalAlignment="Center" HorizontalAlignment="Center">
    <ItemsControl ItemsSource="{Binding FanBarList}">
        <ItemsControl.ItemsPanel>
            <ItemsPanelTemplate>
                <StackPanel Orientation="Vertical" />
            </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <Rectangle Height="10" Width="50" Margin="5"
                           Fill="{Binding}" />
            </DataTemplate>
        </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- 模擬旋鈕轉動的按鈕 -->
    <StackPanel Orientation="Horizontal" Margin="10" HorizontalAlignment="Center">
        <Button Content="⟲" Width="40" Command="{Binding DriverKnobCounterClockwiseCommand}" />
        <Button Content="⟳" Width="40" Command="{Binding DriverKnobClockwiseCommand}" Margin="10,0,0,0" />
    </StackPanel>
</StackPanel>

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

public ObservableCollection<Brush> FanBarList { get; set; } = new ObservableCollection<Brush>();

private ClimateState _state = new ClimateState();
public ClimateState State
{
    get => _state;
    set
    {
        _state = value;
        OnPropertyChanged(nameof(State));
    }
}

public ICommand DriverKnobClockwiseCommand { get; }
public ICommand DriverKnobCounterClockwiseCommand { get; }

public ClimatePageViewModel()
{
    UpdateFanBar();

    DriverKnobClockwiseCommand = new RelayCommand(() =>
    {
        if (State.DriverFanLevel < 5)
        {
            State.DriverFanLevel++;
            UpdateFanBar();
        }
    });

    DriverKnobCounterClockwiseCommand = new RelayCommand(() =>
    {
        if (State.DriverFanLevel > 0)
        {
            State.DriverFanLevel--;
            UpdateFanBar();
        }
    });
}



