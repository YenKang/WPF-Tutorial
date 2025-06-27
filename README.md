private void ProcessEvent(KnobStatus previous, KnobStatus current)
{
    // 🔁 Rotate 行為（觸控中，且 Counter 變動）
    if (current.IsTouched &&
        previous.IsTouched &&
        current.Counter != previous.Counter)
    {
        int delta = current.Counter - previous.Counter;

        // 🔍 印出詳細 Rotate log
        Debug.WriteLine($"🔄 Rotate Detected → KnobId: {current.Id}, Role: {current.Role}, " +
                        $"Counter: {previous.Counter} → {current.Counter}, Delta: {delta}");

        var rotateEvent = KnobEvent.CreateRotate(current.Role, delta);
        _router.Route(rotateEvent);
    }

    // 🔘 Press 行為（從未按下 → 按下）
    if (current.IsTouched &&
        !previous.IsPressed &&
        current.IsPressed)
    {
        // 🔍 印出 Press log
        Debug.WriteLine($"🟢 Press Detected → KnobId: {current.Id}, Role: {current.Role}, Pressed=True");

        var pressEvent = KnobEvent.CreatePress(current.Role);
        _router.Route(pressEvent);
    }
}