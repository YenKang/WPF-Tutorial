private void UpdateGlowStatus(KnobEvent knobEvent)
{
    if (knobEvent.TargetKnob == KnobType.Right)
    {
        if (knobEvent.IsTouched || knobEvent.IsRotated || knobEvent.IsPressed)
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
        // 未來 Left Knob 的 Glow 更新邏輯可放這
    }
}