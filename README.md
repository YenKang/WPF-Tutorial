# WPF-Tutorial

<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="0,20">
    <Button Content="順時針 ➕" 
            Command="{Binding DriverKnobClockwiseCommand}" 
            Width="100" Margin="10"/>
    <Button Content="逆時針 ➖" 
            Command="{Binding DriverKnobCounterClockwiseCommand}" 
            Width="100" Margin="10"/>
</StackPanel>