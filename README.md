private static readonly HashSet<string> AllowedTypes = new HashSet<string>
{
    "grayLevelControl", "cursorControl", "grayReverseControl", "flickerControl"
};

private static void Validate(ProfileRoot root, IDictionary<string,string> registers, List<string> warnings)
{
    if (root.schemaVersion <= 0)
        warnings.Add("schemaVersion is missing or invalid; assuming 1.");

    if (root.patterns == null || root.patterns.Count == 0)
        warnings.Add("No patterns defined.");

    foreach (var p in root.patterns)
    {
        // regControlType 檢查
        if (p.regControlType != null)
        {
            foreach (var t in p.regControlType)
                if (!AllowedTypes.Contains(t))
                    warnings.Add($"Pattern '{p.name}': unknown regControlType '{t}'.");
        }

        // grayLevelControl（若是強型別就檢，否則略過）
        var glc = p.grayLevelControl as Newtonsoft.Json.Linq.JObject;
        if (glc != null)
        {
            // fields
            var fields = glc["fields"] as Newtonsoft.Json.Linq.JObject;
            if (fields != null)
            {
                foreach (var key in new[] { "D2", "D1", "D0" })
                {
                    var f = fields[key] as Newtonsoft.Json.Linq.JObject;
                    if (f == null) { warnings.Add($"Pattern '{p.name}': fields.{key} missing."); continue; }

                    int min = (int?)f["min"] ?? 0;
                    int max = (int?)f["max"] ?? 0;
                    int def = (int?)f["default"] ?? 0;
                    if (min > max) warnings.Add($"Pattern '{p.name}': fields.{key} min>max.");
                    if (def < min || def > max) warnings.Add($"Pattern '{p.name}': fields.{key} default out of range.");
                }
            }

            // writes
            var writes = glc["writes"] as Newtonsoft.Json.Linq.JArray;
            if (writes != null && registers != null)
            {
                foreach (Newtonsoft.Json.Linq.JObject w in writes)
                {
                    var target = (string)w["target"];
                    if (!string.IsNullOrEmpty(target) && !registers.ContainsKey(target))
                        warnings.Add($"Pattern '{p.name}': writes.target '{target}' not found in Registers.");
                }
            }
        }
    }
}