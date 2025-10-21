using System.Collections.ObjectModel; // 別忘了

public sealed class AutoRunVM : ViewModelBase
{
    // 可選的 pattern index 清單（提供給 ORD 的 ComboBox）
    public ObservableCollection<int> PatternIndexOptions { get; }
        = new ObservableCollection<int>();

    // ...其餘既有成員
}