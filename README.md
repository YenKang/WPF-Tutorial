private void LeftKnobOuter_MouseEnter(object sender, MouseEventArgs e)
{
    System.Diagnostics.Debug.WriteLine("⚡ Mouse Enter Triggered");
    LeftKnobGlow.Opacity = 0.6;
}

private void LeftKnobOuter_MouseLeave(object sender, MouseEventArgs e)
{
    System.Diagnostics.Debug.WriteLine("⚡ Mouse Leave Triggered");
    LeftKnobGlow.Opacity = 0;
}