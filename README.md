public string FilterRawDataByPrefix(string rawInput, string prefix)
{
    if (string.IsNullOrWhiteSpace(rawInput))
        return string.Empty;

    var lines = rawInput
        .Split(new[] { '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries)
        .Where(line => line.StartsWith(prefix));

    return string.Join(Environment.NewLine, lines);
}

FilterRawDataByPrefix(rawData, "Knob_");
FilterRawDataByPrefix(rawData, "Finger_");