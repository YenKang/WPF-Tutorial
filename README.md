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

                // 取得舊狀態（若無則初始化為 current 的 clone）
                if (!_knobStatusMap.TryGetValue(current.Id, out var previous))
                    previous = current.Clone();

                // ✨ 將判斷邏輯拆出來
                ProcessEvent(previous, current);

                // 更新狀態快照
                _knobStatusMap[current.Id] = current.Clone();
            }
        }

        private void ProcessEvent(KnobStatus previous, KnobStatus current)
        {
            // ✅ Rotate 判斷條件
            if (current.IsTouched &&
                previous.IsTouched &&
                current.Counter != previous.Counter)
            {
                int delta = current.Counter - previous.Counter;
                var rotateEvent = KnobEvent.CreateRotate(current.Role, delta);
                _router.Route(rotateEvent);
            }

            // ✅ Press 判斷條件（false → true）
            if (current.IsTouched &&
                !previous.IsPressed &&
                current.IsPressed)
            {
                var pressEvent = KnobEvent.CreatePress(current.Role);
                _router.Route(pressEvent);
            }
        }
    }
}