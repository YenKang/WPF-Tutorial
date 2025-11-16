public string FCNT2_Text
{
    get { return FCNT2_Value.ToString("X3"); }
    set
    {
        if (string.IsNullOrWhiteSpace(value))
            return;

        var s = value.Trim();

        if (!int.TryParse(s, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out var parsed))
            return;

        parsed = Clamp(parsed, _min, _max);
        SetFromValue(parsed);
        OnPropertyChanged(nameof(FCNT2_Text));
    }
}