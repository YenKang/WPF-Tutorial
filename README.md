<ItemsControl ItemsSource="{Binding LeftFanBarIndexList}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <!-- 這裡的 DataContext == 當前的 index 值（0~4） -->
            <Button Command="{Binding DataContext.SetLeftFanLevelCmd,
                                      RelativeSource={RelativeSource AncestorType=UserControl}}"
                    CommandParameter="{Binding}">
                <Rectangle Width="60" Height="12" Margin="2" Fill="Gray" />
            </Button>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>


public ICommand SetLeftFanLevelCmd { get; }

public ClimatePageViewModel()
{
    SetLeftFanLevelCmd = new RelayCommand<int>(OnSetLeftFanLevel);
}

private void OnSetLeftFanLevel(int index)
{
    // index 是 0~4
    LeftFanLevel = index + 1; // Level 從 1 開始
    LeftCT = Models.ControlTarget.LeftFan;
}

public IList<int> LeftFanBarIndexList { get; } = new List<int> { 0, 1, 2, 3, 4 };