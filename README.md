public class ClimatePageViewModel : IKnobHandler
{
    public int LeftTemperature { get; set; } = 22;
    public int RightTemperature { get; set; } = 22;

    public void OnDriverKnobRotated(KnobEvent e)
    {
        LeftTemperature += e.Delta;
        Debug.WriteLine($"🌡 主駕旋鈕調整溫度：{LeftTemperature}");
        OnPropertyChanged(nameof(LeftTemperature));
    }

    public void OnPassengerKnobRotated(KnobEvent e)
    {
        RightTemperature += e.Delta;
        Debug.WriteLine($"🌡 副駕旋鈕調整溫度：{RightTemperature}");
        OnPropertyChanged(nameof(RightTemperature));
    }

    public void OnDriverKnobPressed(KnobEvent e)
    {
        Debug.WriteLine($"🟢 主駕按下旋鈕");
    }

    public void OnPassengerKnobPressed(KnobEvent e)
    {
        Debug.WriteLine($"🟢 副駕按下旋鈕");
    }
}