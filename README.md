
```csharp
using System;
using System.Collections.Generic;

namespace NovaCID.Knob
{
    public class KnobEventProcessor
    {
        private readonly Dictionary<string, KnobStatus> _knobStatusMap = new();
        private readonly IKnobEventRouter _router;

        public KnobEventProcessor(IKnobEventRouter router)
        {
            _router = router;
        }

        public void ParseAndProcess(string rawData)
        {
            var lines = rawData.Split(new[] { '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);

            foreach (var line in lines)
            {
                var newStatus = KnobStatus.Parse(line);

                // 比對前一筆狀態（如果有的話）
                if (_knobStatusMap.TryGetValue(newStatus.Id, out var oldStatus))
                {
                    ProcessEvents(oldStatus, newStatus);
                }

                // 更新快照
                _knobStatusMap[newStatus.Id] = new KnobStatus
                {
                    Id = newStatus.Id,
                    Role = newStatus.Role,
                    IsTouch = newStatus.IsTouch,
                    Counter = newStatus.Counter,
                    Press = newStatus.Press,
                    PreviousTouch = newStatus.IsTouch,    // 這些會在下次當作 "oldStatus" 使用
                    PreviousCounter = newStatus.Counter,
                    PreviousPress = newStatus.Press
                };
            }
        }
    }
}
```

## Compare Logic

```
private void ProcessEvents(KnobStatus oldStatus, KnobStatus newStatus)
{
	// ✅ Rotate 定義：Touch 是 true，且 Counter 有變化
	if (newStatus.IsTouch && newStatus.Counter != oldStatus.Counter)
	{
		int delta = newStatus.Counter - oldStatus.Counter;
		var rotateEvent = KnobEvent.CreateRotate(newStatus.Role, delta);
		_router.Route(rotateEvent);
	}

	// ✅ Press 定義：Touch 為 true 且 Press: false ➜ true
	if (newStatus.IsTouch && !oldStatus.Press && newStatus.Press)
	{
		var pressEvent = KnobEvent.CreatePress(newStatus.Role);
		_router.Route(pressEvent);
	}
}

```
