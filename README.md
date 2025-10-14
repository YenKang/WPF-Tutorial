// ==========================
//  BistModeViewModel.cs
// ==========================
private IcProfile _profile;
private bool _busReady = false;
private bool _profileReady = false;

public RegisterReadWriteEx RegDisplayControl { get; private set; }

// ---------- SetRegDisplay (由外部呼叫) ----------
public void SetRegDisplay(object RegDisplayObj)
{
    var reg = RegDisplayObj as RegisterReadWriteEx;
    if (reg == null) return;

    RegDisplayControl = reg;

    // 包裝成 IRegisterBus adapter
    var bus = new RegDisplayBus(RegDisplayControl);
    RegMap.Init(bus);
    _busReady = true;

    // 檢查：若 profile 已載好 → 就可直接完成整合
    TryInitRegMap();
}

// ---------- 載入 Profile ----------
private void LoadProfileFromFile()
{
    var baseDir = AppDomain.CurrentDomain.BaseDirectory;
    var profilePath = System.IO.Path.Combine(baseDir, "Profiles", "NT51365.profile.json");

    _profile = ProfileLoader.Load(profilePath);
    _profileReady = true;

    // 餵給 UI
    Patterns.Clear();
    foreach (var p in _profile.Patterns)
        Patterns.Add(new BistMode.Models.PatternItem { Index = p.Index, Name = p.Name, Icon = p.Icon });

    // 檢查：若 bus 已準備 → 就能立即載入
    TryInitRegMap();
}

// ---------- 雙 Ready 檢查 ----------
private void TryInitRegMap()
{
    if (_busReady && _profileReady)
    {
        RegMap.LoadFrom(_profile);
        // 這裡可加 DebugLog("RegMap initialized");
    }
}
