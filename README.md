<Button Content="Test RawCounter 10" Click="TestRaw10_Click" Width="250" Height="60" Margin="20"/>
<Button Content="Test RawCounter 128" Click="TestRaw128_Click" Width="250" Height="60" Margin="20"/>
<Button Content="Test RawCounter 200" Click="TestRaw200_Click" Width="250" Height="60" Margin="20"/>

private void TestRaw10_Click(object sender, RoutedEventArgs e)
{
    UpdateRightKnobArc(10);
}

private void TestRaw128_Click(object sender, RoutedEventArgs e)
{
    UpdateRightKnobArc(128);
}

private void TestRaw200_Click(object sender, RoutedEventArgs e)
{
    UpdateRightKnobArc(200);
}