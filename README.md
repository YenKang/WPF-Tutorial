private void AdjustFCNT2(int delta)
{
    var v = FCNT2_Value + delta;
    v = Clamp(v, _min, _max);
    SetFromValue(v);
    OnPropertyChanged(nameof(FCNT2_Text));
}