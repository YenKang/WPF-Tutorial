namespace NovaCID.Knob
{
    public enum KnobId
    {
        Left,
        Right
    }

    public static class KnobIdHelper
    {
        public static KnobId ParseFromString(string knobString)
        {
            if (string.IsNullOrWhiteSpace(knobString))
                throw new ArgumentException("knobString is null or empty");

            knobString = knobString.Trim();

            return knobString switch
            {
                "Knob_0" => KnobId.Left,
                "Knob_1" => KnobId.Right,
                _ => throw new ArgumentException($"未知的 Knob ID：{knobString}")
            };
        }
    }
}