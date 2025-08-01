public class DriveModeState : INotifyPropertyChanged
{
    private double _zoomLevel = 0;

    public double ZoomLevel
    {
        get => _zoomLevel;
        set
        {
            if (_zoomLevel != value)
            {
                _zoomLevel = value;
                OnPropertyChanged(nameof(ZoomLevel));
            }
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void OnPropertyChanged(string propName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propName));
    }
}

private DriveModeState _state = new DriveModeState();
public DriveModeState State => _state;

public double ZoomLevel
{
    get => State.ZoomLevel;
    set
    {
        if (State.ZoomLevel != value)
        {
            State.ZoomLevel = value;
            OnPropertyChanged(nameof(ZoomLevel));  // 通知 UI
            UpdateCurrentMap();                    // 附加邏輯
        }
    }
}