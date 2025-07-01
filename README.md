private void LeftKnobOuter_MouseEnter(object sender, MouseEventArgs e)
{
    LeftKnobGlow.Opacity = 0.6; // 顯示 Glow，亮度你可自己調整
}

private void LeftKnobOuter_MouseLeave(object sender, MouseEventArgs e)
{
    LeftKnobGlow.Opacity = 0; // 滑鼠離開時關閉 Glow
}