

public class FingerStatus
{
    public int FingerId { get; set; }
    public bool IsDriver { get; set; }
    public string Status { get; set; } // 目前是 None
    public int X { get; set; }
    public int Y { get; set; }

    public static FingerStatus Parse(string line)
    {
        // 範例行：Finger_0, Driver=False, Status=None, X=1346, Y=1046
        var parts = line.Split(',');

        return new FingerStatus
        {
            FingerId = int.Parse(parts[0].Split('_')[1]),
            IsDriver = parts[1].Split('=')[1].Trim() == "True",
            Status = parts[2].Split('=')[1].Trim(),
            X = int.Parse(parts[3].Split('=')[1]),
            Y = int.Parse(parts[4].Split('=')[1])
        };
    }
}