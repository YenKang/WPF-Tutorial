using System.Text.Json;

public sealed class IcProfile
{
    public string IcName { get; set; } = "";
    public Dictionary<string,string> Registers { get; set; } = new();
    public List<PatternItem> Patterns { get; set; } = new();
    public GrayControls GrayControls { get; set; } = new();
}

public sealed class PatternItem
{
    public int Index { get; set; }
    public string Name { get; set; } = "";
    public string? Icon { get; set; }
}

public sealed class GrayControls
{
    public List<GrayTarget> Targets { get; set; } = new();
}

public sealed class GrayTarget
{
    public string Id { get; set; } = "";      // "PT_LEVEL"
    public string Name { get; set; } = "";    // "Pattern Level"
    public Range Range { get; set; } = new(); // {min,max}
    public List<int>? ValidPatterns { get; set; }
}

public sealed class Range { public int Min { get; set; } public int Max { get; set; } }

public static class ProfileLoader
{
    public static IcProfile Load(string path) =>
        JsonSerializer.Deserialize<IcProfile>(File.ReadAllText(path),
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true })!;
}
