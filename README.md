var path = @"Assets/Profiles/NT51365.profile.json"; // 改成你的實際路徑
var p = ProfileLoader.LoadChipProfile(path);

// 簡單驗證
System.Diagnostics.Debug.WriteLine("chip=" + p.chip);
System.Diagnostics.Debug.WriteLine("reg0030=" + p.registers["BIST_PT_LEVEL"].addr);
System.Diagnostics.Debug.WriteLine("pattern=" + p.patterns[0].name);
System.Diagnostics.Debug.WriteLine("row=" + p.patterns[0].rows[0].title);