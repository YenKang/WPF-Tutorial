// /Finger/FingerEventProcessor.cs
using System.Collections.Generic;

public class FingerEventProcessor
{
    private readonly Dictionary<string, FingerStatus> _fingerStatusMap = new();

    public List<FingerStatus> Parse(string rawData)
    {
        var result = new List<FingerStatus>();
        if (string.IsNullOrWhiteSpace(rawData))
            return result;

        string[] lines = rawData.Split(new[] { '\r', '\n' }, System.StringSplitOptions.RemoveEmptyEntries);

        foreach (var line in lines)
        {
            if (line.StartsWith("Finger_"))
            {
                var finger = FingerStatus.Parse(line);
                result.Add(finger);

                // 可以選擇這邊做後續狀態記憶與比較
                _fingerStatusMap[finger.Id] = finger.Clone(); // 或後續判斷差異再處理
            }
        }

        return result;
    }
}