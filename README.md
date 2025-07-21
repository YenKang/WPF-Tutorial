public void OnKnobRotated(KnobRole role, int delta)
{
    if (role != KnobRole.Driver) return;

    switch (LeftControlTarget)
    {
        case LeftControlTarget.Temperature:
            LeftTemperature += delta * 0.5; // 每格調整 0.5°C
            break;

        case LeftControlTarget.Fan:
            int newLevel = LeftFanLevel + delta;
            if (newLevel >= 0 && newLevel <= 5)
                LeftFanLevel = newLevel;
            break;
    }
}