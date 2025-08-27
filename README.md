using System;
using System.Windows.Threading;

sealed class GlowDebouncer
{
    private readonly DispatcherTimer _timer;
    private readonly Action _onTimeout;

    public GlowDebouncer(TimeSpan timeout, Action onTimeout)
    {
        _onTimeout = onTimeout;
        _timer = new DispatcherTimer(DispatcherPriority.Background, System.Windows.Application.Current.Dispatcher)
        {
            Interval = timeout
        };
        _timer.Tick += (s, e) => { _timer.Stop(); _onTimeout(); };
    }

    public void Bump()    { _timer.Stop(); _timer.Start(); }
    public void StopNow() { _timer.Stop(); _onTimeout(); }
}

