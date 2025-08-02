```CSHARP
public class DrivePageViewModel : INotifyPropertyChanged
{
    public DriveState State { get; set; } = new DriveState();

    public double ZoomLevel
    {
        get => State.ZoomLevel;
        set
        {
            if (State.ZoomLevel != value)
            {
                State.ZoomLevel = value;
                OnPropertyChanged(); // 通知 View 更新
            }
        }
    }

    public ICommand ZoomInCommand { get; }

    public DrivePageViewModel()
    {
        ZoomInCommand = CommandFactory.CreateCommand(ZoomIn);
    }

    public void ZoomIn() => ZoomLevel += 0.1;

    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
      => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```
