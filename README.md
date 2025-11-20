<!-- FCNT1 -->
<StackPanel Orientation="Horizontal" Margin="0,12,0,4">
    <TextBlock Text="FCNT1:"
               VerticalAlignment="Center"
               Margin="0,0,4,0"/>

    <TextBlock Text="0x"
               VerticalAlignment="Center"
               Margin="0,0,4,0"/>

    <xctk:IntegerUpDown Width="70"
                        FormatString="X3"
                        Value="{Binding FCNT1_Value,
                                        Mode=TwoWay,
                                        UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>

<!-- FCNT2 -->
<StackPanel Orientation="Horizontal" Margin="0,0,0,8">
    <TextBlock Text="FCNT2:"
               VerticalAlignment="Center"
               Margin="0,0,4,0"/>

    <TextBlock Text="0x"
               VerticalAlignment="Center"
               Margin="0,0,4,0"/>

    <xctk:IntegerUpDown Width="70"
                        FormatString="X3"
                        Value="{Binding FCNT2_Value,
                                        Mode=TwoWay,
                                        UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>
