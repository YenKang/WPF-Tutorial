private void LoadProfileFromFile(string icName = "NT51365")
{
    try
    {
        // 1) 找 Profile 檔案
        var baseDir     = AppDomain.CurrentDomain.BaseDirectory;
        var profilesDir = System.IO.Path.Combine(baseDir, "Profiles");
        var profilePath = System.IO.Path.Combine(profilesDir, icName + ".profile.json");

        if (!System.IO.File.Exists(profilePath))
        {
            System.Windows.MessageBox.Show("找不到 Profile 檔案：\n" + profilePath);
            return;
        }

        // 2) 載入 JSON → _profile
        _profile = ProfileLoader.Load(profilePath);
        _profileReady = true;

        // 3) 根據 icName 組圖檔基底（Site-of-Origin）
        //    圖檔請放：Assets/Patterns/<icName>/，並設 Content + Copy if newer
        string baseUri = "pack://siteoforigin:,,,/Assets/Patterns/" + _profile.IcName + "/";

        // 4) 餵 UI：把每筆 pattern 的 icon 補成完整 URI
        Patterns.Clear();
        foreach (var p in _profile.Patterns)
        {
            string iconUri = null;
            if (!string.IsNullOrEmpty(p.Icon))
            {
                var filename = p.Icon.TrimStart('/', '\\');      // 僅檔名或相對路徑
                iconUri = baseUri + filename;                    // 完整 pack URI
            }

            Patterns.Add(new BistMode.Models.PatternItem
            {
                Index = p.Index,
                Name  = p.Name,
                Icon  = iconUri
            });
        }

        // 5) 嘗試初始化 RegMap（若 bus 已就緒會直接成功）
        TryInitRegMap();

        System.Diagnostics.Debug.WriteLine("✅ Profile 載入完成：" + _profile.IcName +
            "，Patterns=" + _profile.Patterns.Count);
    }
    catch (System.Exception ex)
    {
        System.Windows.MessageBox.Show("載入 Profile 失敗：\n" + ex.Message);
    }
}
