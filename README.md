using System.Collections.ObjectModel;
using System.ComponentModel;

namespace BistApp.ViewModels
{
    public class PatternItem : INotifyPropertyChanged
    {
        public string Id { get; set; } = "";      // P0, P1...
        public string Name { get; set; } = "";    // White, Black...
        public string Icon { get; set; } = "";    // 資料夾相對路徑
        public string DisplayName => $"{Id} {Name}";
        public event PropertyChangedEventHandler? PropertyChanged;
    }

    public class MainWindowViewModel
    {
        public ObservableCollection<PatternItem> Patterns { get; } = new();

        public MainWindowViewModel()
        {
            // 先放假資料：等會兒就能看到畫面
            Patterns.Add(new PatternItem { Id="P0", Name="White", Icon="Assets/Patterns/white.png" });
            Patterns.Add(new PatternItem { Id="P1", Name="Black", Icon="Assets/Patterns/black.png" });
            Patterns.Add(new PatternItem { Id="P2", Name="Red",   Icon="Assets/Patterns/red.png" });
            Patterns.Add(new PatternItem { Id="P3", Name="Green", Icon="Assets/Patterns/green.png" });
            Patterns.Add(new PatternItem { Id="P4", Name="Blue",  Icon="Assets/Patterns/blue.png" });
        }
    }
}