public class PatternItem
{
    public string Name { get; set; } = "";
    public string Icon { get; set; } = "";
}

public class BistModeViewModel
{
    public ObservableCollection<PatternItem> Patterns { get; } = new();

    public BistModeViewModel()
    {
        Patterns.Add(new PatternItem { Name="Black", Icon="pack://siteoforigin:,,,/Assets/Patterns/NT51635/PatIdx_0_Black.png" });
        Patterns.Add(new PatternItem { Name="White", Icon="pack://siteoforigin:,,,/Assets/Patterns/NT51635/PatIdx_1_White.png" });
    }
}