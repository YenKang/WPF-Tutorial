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

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
<ListView ItemsSource="{Binding Patterns}" Margin="10" BorderThickness="0" Background="Transparent">
  <ListView.ItemsPanel>
    <ItemsPanelTemplate><WrapPanel/></ItemsPanelTemplate>
  </ListView.ItemsPanel>

  <ListView.ItemTemplate>
    <DataTemplate>
      <StackPanel Width="140" Margin="6">
        <Image Source="{Binding Icon}" Width="120" Height="70" Stretch="UniformToFill"/>
        <TextBlock Text="{Binding Name}" HorizontalAlignment="Center" Margin="0,6,0,0"/>
      </StackPanel>
    </DataTemplate>
  </ListView.ItemTemplate>
</ListView>