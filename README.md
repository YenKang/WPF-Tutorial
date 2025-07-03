<StackPanel Orientation="Vertical" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="30">

    <!-- X 軸控制 -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,5">
        <TextBlock Text="X:" Foreground="White"/>
        <TextBox x:Name="LeftKnob_TextX" Width="60" Text="445" Margin="5" TextChanged="LeftKnob_TextX_TextChanged"/>
    </StackPanel>
    <Slider x:Name="LeftKnob_SliderX" Minimum="0" Maximum="2560" Value="445" Width="200" ValueChanged="LeftKnob_Slider_ValueChanged"/>

    <!-- Y 軸控制 -->
    <StackPanel Orientation="Horizontal" Margin="0,10,0,5">
        <TextBlock Text="Y:" Foreground="White"/>
        <TextBox x:Name="LeftKnob_TextY" Width="60" Text="1113" Margin="5" TextChanged="LeftKnob_TextY_TextChanged"/>
    </StackPanel>
    <Slider x:Name="LeftKnob_SliderY" Minimum="0" Maximum="1440" Value="1113" Width="200" ValueChanged="LeftKnob_Slider_ValueChanged"/>

</StackPanel>

