```csharp
using System;
using System.Collections.Generic;

namespace NovaCID.Knob
{
    public class KnobEventProcessor
    {
        private readonly IKnobEventRouter _router;
        private readonly Dictionary<string, KnobStatus> _knobStatusMap;

        public KnobEventProcessor(IKnobEventRouter router)
        {
            _router = router ?? throw new ArgumentNullException(nameof(router));
            _knobStatusMap = new Dictionary<string, KnobStatus>();
        }

        public void ParseAndProcess(string rawData)
        {
            if (string.IsNullOrWhiteSpace(rawData))
                return;

            string[] lines = rawData.Split(new[] { '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);

            foreach (string line in lines)
            {
                KnobStatus current = KnobStatus.Parse(line);

                // 取出上一個狀態，沒有的話用目前的初始
                if (!_knobStatusMap.TryGetValue(current.Id, out var previous))
                    previous = current.Clone();

                // 記錄前一次狀態，用來比對
                current.PreviousTouched = previous.IsTouched;
                current.PreviousPressed = previous.IsPressed;
                current.PreviousCounter = previous.Counter;

                // 🎯 判斷 Rotate 行為
                if (current.IsTouched &&
                    current.PreviousTouched &&
                    current.Counter != current.PreviousCounter)
                {
                    int delta = current.Counter - current.PreviousCounter;
                    var rotateEvent = KnobEvent.CreateRotate(current.Role, delta);
                    _router.Route(rotateEvent);
                }

                // 🎯 判斷 Press 行為（需 Press 從 false → true）
                if (current.IsTouched &&
                    !current.PreviousPressed &&
                    current.IsPressed)
                {
                    var pressEvent = KnobEvent.CreatePress(current.Role);
                    _router.Route(pressEvent);
                }

                // 更新當前狀態
                _knobStatusMap[current.Id] = current.Clone();
            }
        }
    }
}
```


public KnobStatus Clone()
{
    return new KnobStatus
    {
        Id = this.Id,
        Role = this.Role,
        IsTouched = this.IsTouched,
        Counter = this.Counter,
        IsPressed = this.IsPressed,
        PreviousTouched = this.PreviousTouched,
        PreviousCounter = this.PreviousCounter,
        PreviousPressed = this.PreviousPressed
    };
}