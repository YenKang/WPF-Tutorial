using System.IO;
using System.Xml.Linq;

public static class SettingsLoader
{
    public static AppSettings Load(string filePath)
    {
        if (!File.Exists(filePath)) return new AppSettings();

        var xml = XElement.Load(filePath);
        return new AppSettings
        {
            KnobEnabled = bool.TryParse(xml.Element("KnobEnabled")?.Value, out var result) && result
        };
    }
}