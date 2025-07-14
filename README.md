<StackPanel Orientation="Vertical" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="0,20,0,0">
    <Button
        Content="➕ Increase Right Fan"
        Command="{Binding IncreaseRightFanLevelCommand}"
        Width="220"
        Height="60"
        Margin="0,10,0,0"/>

    <Button
        Content="➖ Decrease Right Fan"
        Command="{Binding DecreaseRightFanLevelCommand}"
        Width="220"
        Height="60"
        Margin="0,10,0,0"/>
</StackPanel>


－－－

```chsarp
public ICommand IncreaseRightFanLevelCommand => new RelayCommand(() =>
{
    if (RightFanLevel < 5)
        RightFanLevel++;
});

public ICommand DecreaseRightFanLevelCommand => new RelayCommand(() =>
{
    if (RightFanLevel > 0)
        RightFanLevel--;
});
```