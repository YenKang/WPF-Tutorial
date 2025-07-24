<Button Content="Zoom In"
        Command="{Binding ZoomInCommand}"
        Visibility="{Binding KnobEnabled, Converter={StaticResource InvertBoolToVis}}"/>


using System.Windows;
using NovaCID.ViewModel; // 確保有引入你的主 VM 命名空間

public class DrivePageViewModel : INotifyPropertyChanged
{
    // 其他屬性、建構子...

    // === [KnobEnabled 跨 ViewModel 屬性] ===
    public bool KnobEnabled => GetKnobEnabledFromMainVM();

    private bool GetKnobEnabledFromMainVM()
    {
        return (App.Current.MainWindow.DataContext as NovaCIDViewModel)?.KnobEnabled ?? true;
    }

    // INotifyPropertyChanged 實作等...
}