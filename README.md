private IcProfile _selectedProfile = IcProfile.Primary;
public IcProfile SelectedProfile
{
    get { return _selectedProfile; }
    set
    {
        if (_selectedProfile != value)
        {
            _selectedProfile = value;
            RaisePropertyChanged(nameof(SelectedProfile));
        }
    }
}