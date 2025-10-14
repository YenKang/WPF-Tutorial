using System;
using System.Collections.Generic;
using System.IO;
using System.Text.Json;

public sealed class IcProfile
{
    public IcProfile()
    {
        Registers = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
        Patterns = new List<PatternItem>();
        GrayControls = new GrayControls();
    }

    public string IcName { get; set; }
    public Dictionary<string, string> Registers { get; set; }
    public List<PatternItem> Patterns { get; set; }
    public GrayControls GrayControls { get; set; }
}

public sealed class PatternItem
{
    public PatternItem() { }

    public int Index { get; set; }
    public string Name { get; set; }
    public string Icon { get; set; } // 允許為 null（C#7.3 沒可空註記，保留為 string）
}

public sealed class GrayControls
{
    public GrayControls()
    {
        Targets = new List<GrayTarget>();
    }

    public List<GrayTarget> Targets { get; set; }
}

public sealed class GrayTarget
{
    public GrayTarget()
    {
        Range = new Range();
        ValidPatterns = new List<int>();
    }

    public string Id { get; set; }      // "PT_LEVEL"
    public string Name { get; set; }    // "Pattern Level"
    public Range Range { get; set; }    // {min,max}
    public List<int> ValidPatterns { get; set; } // 可為空集合表示不限制
}

public sealed class Range
{
    public int Min { get; set; }
    public int Max { get; set; }
}

public static class ProfileLoader
{
    public static IcProfile Load(string path)
    {
        if (!File.Exists(path))
            throw new FileNotFoundException("Profile json not found.", path);

        var json = File.ReadAllText(path);
        var options = new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        };
        var profile = JsonSerializer.Deserialize<IcProfile>(json, options);
        if (profile == null)
            throw new InvalidDataException("Failed to deserialize profile.");

        // 防禦性補位，避免 null 參考
        if (profile.Registers == null)
            profile.Registers = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
        if (profile.Patterns == null)
            profile.Patterns = new List<PatternItem>();
        if (profile.GrayControls == null)
            profile.GrayControls = new GrayControls();
        if (profile.GrayControls.Targets == null)
            profile.GrayControls.Targets = new List<GrayTarget>();

        return profile;
    }
}
