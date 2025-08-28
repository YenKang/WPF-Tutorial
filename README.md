public bool LeftKnobGlowOn
{
    get => _leftKnobGlowOn;
    set
    {
        if (_leftKnobGlowOn == value) return;
        _leftKnobGlowOn = value;
        Console.WriteLine($"[{DateTime.Now:HH:mm:ss.fff}] LeftKnobGlowOn = {value}");
        OnPropertyChanged();
    }
}