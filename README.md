public class BistModeViewModel : ViewModelBase
{
    public ObservableCollection<PatternItem> Patterns { get; } 
        = new ObservableCollection<PatternItem>();

    public PatternItem SelectedPattern
    {
        get => _selectedPattern;
        set
        {
            if (_selectedPattern == value) return;
            _selectedPattern = value;
            SetValue(value);
            ApplyPatternToGroups(_selectedPattern);
        }
    }

    public AutoRunVM AutoRunVM { get; } = new AutoRunVM();
    public bool IsAutoRunConfigVisible
    {
        get => GetValue<bool>();
        set => SetValue(value);
    }

    // 🟩 這是你的 Profile 載入方法
    public void LoadProfileFromFile(string path)
    {
        _profile = JsonConfigUiRunner.LoadRoot(path);

        Patterns.Clear();
        foreach (var node in _profile.Patterns)
        {
            Patterns.Add(new PatternItem
            {
                Index = node.index,
                Name = node.name,
                IconUri = ResolveIcon(node.icon),
                RegControlType = node.regControlType ?? new List<string>(),
                AutoRunControl = node.autoRunControl
            });
        }

        // ✅ 在這裡呼叫 AfterProfileLoaded()
        AfterProfileLoaded();
    }

    // 🟩 這是你的 ApplyPatternToGroups()
    private void ApplyPatternToGroups(PatternItem p)
    {
        var types = p?.RegControlType ?? new List<string>();

        // AutoRun 顯示條件
        IsAutoRunConfigVisible = types.Any(t =>
            string.Equals(t?.Trim(), "autoRunControl", StringComparison.OrdinalIgnoreCase));

        if (IsAutoRunConfigVisible && p?.AutoRunControl != null)
            AutoRunVM.LoadFrom((JObject)p.AutoRunControl);
        else
            IsAutoRunConfigVisible = false;
    }

    // 🟩 ✅ 把 AfterProfileLoaded 放在這裡（class 內）
    private void AfterProfileLoaded()
    {
        // 1️⃣ 供 AutoRun 使用的 pattern index 選項
        AutoRunVM.PatternIndexOptions.Clear();
        foreach (var p in Patterns.OrderBy(x => x.Index))
            AutoRunVM.PatternIndexOptions.Add(p.Index);

        // 2️⃣ 預設隱藏 AutoRun 區塊
        IsAutoRunConfigVisible = false;
    }
}