using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;

namespace OSDIconFlashMap.Util
{
    public static class IniHelper
    {
        public static Dictionary<string, Dictionary<string, string>> ReadAsSections(string path)
        {
            var result = new Dictionary<string, Dictionary<string, string>>(StringComparer.OrdinalIgnoreCase);
            if (!File.Exists(path)) return result;

            string current = null;
            foreach (var raw in File.ReadAllLines(path))
            {
                var line = raw.Trim();
                if (line.Length == 0 || line.StartsWith(";")) continue;

                if (line.StartsWith("[") && line.EndsWith("]"))
                {
                    current = line.Substring(1, line.Length - 2).Trim();
                    if (!result.ContainsKey(current))
                        result[current] = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
                    continue;
                }

                int eq = line.IndexOf('=');
                if (eq > 0 && current != null)
                {
                    var k = line.Substring(0, eq).Trim();
                    var v = line.Substring(eq + 1).Trim();
                    result[current][k] = v;
                }
            }
            return result;
        }

        public static void WriteSections(string path, Dictionary<string, Dictionary<string, string>> data)
        {
            var sb = new StringBuilder();

            if (data.ContainsKey("Meta"))
            {
                sb.AppendLine("[Meta]");
                foreach (var kv in data["Meta"])
                    sb.AppendLine($"{kv.Key}={kv.Value}");
                sb.AppendLine();
            }

            foreach (var sec in data.Keys.Where(k => !string.Equals(k, "Meta", StringComparison.OrdinalIgnoreCase)))
            {
                sb.AppendLine($"[{sec}]");
                foreach (var kv in data[sec].OrderBy(kv => kv.Key))
                    sb.AppendLine($"{kv.Key}={kv.Value}");
                sb.AppendLine();
            }

            File.WriteAllText(path, sb.ToString());
        }
    }
}