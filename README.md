```csharp
public void OnDriverKnobRotated(KnobEvent e)
{
    HandleKnobRotated(e);
}

public void OnPassengerKnobRotated(KnobEvent e)
{
    HandleKnobRotated(e);
}

private void HandleKnobRotated(KnobEvent e)
{
    int delta = ComputeDelta(e.RawCounter, _previousRawCounter);
    _previousRawCounter = e.RawCounter;

    double tempChange = delta * 0.5;

    if (e.KnobSide == Knob.Enums.KnobId.Left)
    {
        LeftTemperature = Clamp(LeftTemperature + tempChange, min_TempLimit, max_TempLimit);
        Debug.WriteLine($"旋鈕調整左側溫度：{LeftTemperature}");
    }
    else if (e.KnobSide == Knob.Enums.KnobId.Right)
    {
        RightTemperature = Clamp(RightTemperature + tempChange, min_TempLimit, max_TempLimit);
        Debug.WriteLine($"旋鈕調整右側溫度：{RightTemperature}");
    }
}
````