<Button 
    Content="模擬右 Knob Increase Temp" 
    Command="{Binding SimulateRightKnobIncreaseTempCommand}" 
    Width="220" 
    Height="50" 
    Margin="10"/>


－－
```csharp
using System.Windows.Input;

namespace YourProjectNamespace.ViewModels
{
    public class ClimatePageViewModel : BaseViewModel
    {
        private int _rightTemperature = 22;
        public int RightTemperature
        {
            get => _rightTemperature;
            set
            {
                if (_rightTemperature != value)
                {
                    _rightTemperature = value;
                    OnPropertyChanged(nameof(RightTemperature));
                }
            }
        }

        private readonly KnobEventRouter _knobEventRouter;

        public ClimatePageViewModel()
        {
            _knobEventRouter = new KnobEventRouter();
        }

        public ICommand SimulateRightKnobIncreaseTempCommand => new RelayCommand(() =>
        {
            var knobEvent = new KnobEvent
            {
                TargetKnob = KnobType.Right,
                EventType = KnobEventType.Rotate,
                Role = KnobRole.Passenger,
                IsRotated = true
            };

            _knobEventRouter.Route(knobEvent);

            RightTemperature++;
        });
    }
}
```