private FileSystemWatcher _watcher;
private string _currentImagePath;
private readonly TimeSpan _debounce = TimeSpan.FromMilliseconds(150);
private DateTime _lastReload = DateTime.MinValue;

private void StartWatch(string fullPath)
{
    StopWatch();

    var dir  = Path.GetDirectoryName(fullPath)!;
    var file = Path.GetFileName(fullPath);
    _currentImagePath = fullPath;

    _watcher = new FileSystemWatcher(dir, file)
    {
        NotifyFilter = NotifyFilters.LastWrite | NotifyFilters.Size | NotifyFilters.FileName | NotifyFilters.CreationTime,
        IncludeSubdirectories = false,
        EnableRaisingEvents = true
    };

    FileSystemEventHandler handler = async (s, e) =>
    {
        var now = DateTime.UtcNow;                         // 去抖
        if (now - _lastReload < _debounce) return;
        _lastReload = now;

        await Task.Delay(120);                             // 等外部寫完
        if (Application.Current != null)
            await Application.Current.Dispatcher.InvokeAsync(() =>
            {
                if (File.Exists(_currentImagePath)) LoadImageFromFile(_currentImagePath);
            });
        else
            if (File.Exists(_currentImagePath)) LoadImageFromFile(_currentImagePath);
    };

    _watcher.Changed += handler;
    _watcher.Created += handler;
    _watcher.Renamed += (s, e) => handler(s, e);
}

private void StopWatch()
{
    if (_watcher == null) return;
    _watcher.EnableRaisingEvents = false;
    _watcher.Dispose();
    _watcher = null;
}