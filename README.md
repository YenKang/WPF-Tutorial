private void UpdateGlowStatus(KnobEvent knobEvent)
{
    if (knobEvent.TargetKnob == KnobType.Right)
    {
        if (knobEvent.IsTouched)
        {
            KnobGlowStatus.Instance.RightKnobRole = knobEvent.Role;
        }
        else
        {
            KnobGlowStatus.Instance.RightKnobRole = KnobRole.None;
        }
    }
    else if (knobEvent.TargetKnob == KnobType.Left)
    {
        // 未來擴充 Left Knob，可寫類似邏輯
    }
}