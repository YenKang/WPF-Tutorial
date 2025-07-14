<Button
    Content="模擬右 Knob Rotate"
    Command="{Binding SimulateRightKnobRawDataCommand}"
    Width="220"
    Height="50"
    Margin="10"/>

111


public ICommand SimulateRightKnobRawDataCommand => new RelayCommand(() =>
{
    // 第 1 筆（初始化）
    string rawData1 = "Knob_1, Driver=True, Touch=True, Press=False, Counter=1";
    var events1 = _knobEventProcessor.Parse(rawData1);
    foreach (var e in events1)
    {
        _knobEventRouter.Route(e);
    }

    // 第 2 筆（觸發 Rotate）
    string rawData2 = "Knob_1, Driver=True, Touch=True, Press=False, Counter=2";
    var events2 = _knobEventProcessor.Parse(rawData2);
    foreach (var e in events2)
    {
        _knobEventRouter.Route(e);
    }
});