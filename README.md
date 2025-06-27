using System;

namespace NovaCID.Knob
{
    public class KnobStatus
    {
        public string Id { get; set; }                 // e.g. "Knob_0" or "Knob_1"
        public KnobRole Role { get; set; }             // Driver / Passenger
        public bool IsTouch { get; set; }              // 當前是否觸控中
        public int Counter { get; set; }               // 當前 Counter 值
        public bool Press { get; set; }                // 當前是否按下

        // ✅ 用於事件偵測的上一次狀態快照（由 KnobEventProcessor 更新）
        public bool PreviousTouch { get; set; }
        public int PreviousCounter { get; set; }
        public bool PreviousPress { get; set; }

        /// <summary>
        /// 從 raw data line 建立新的 KnobStatus 實例
        /// </summary>
        public static KnobStatus Parse(string line)
        {
            var parts = line.Split(',');
            var status = new KnobStatus();

            foreach (var part in parts)
            {
                if (part.StartsWith("Knob_"))
                {
                    status.Id = part.Trim();  // Knob_0 / Knob_1
                }
                else if (part.StartsWith("Role="))
                {
                    var roleStr = part.Substring("Role=".Length);
                    status.Role = roleStr == "Driver" ? KnobRole.Driver : KnobRole.Passenger;
                }
                else if (part.StartsWith("Touch="))
                {
                    var value = part.Substring("Touch=".Length);
                    status.IsTouch = bool.Parse(value);
                }
                else if (part.StartsWith("Counter="))
                {
                    var value = part.Substring("Counter=".Length);
                    status.Counter = int.Parse(value);
                }
                else if (part.StartsWith("Press="))
                {
                    var value = part.Substring("Press=".Length);
                    status.Press = bool.Parse(value);
                }
            }

            return status;
        }

        public override string ToString()
        {
            return $"{Id} ({Role}) | Touch: {IsTouch}, Counter: {Counter}, Press: {Press}";
        }
    }

    public enum KnobRole
    {
        Driver,
        Passenger
    }
}