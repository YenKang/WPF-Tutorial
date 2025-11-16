<GroupBox Header="Black White Loop"
         Margin="0,8,0,0"
         Visibility="{Binding IsBWLoopConfigVisible, Converter={utilityConv:BoolToVisibilityConverter}}">
    <StackPanel Margin="10">
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="FCNT2:"
                       Width="80"
                       VerticalAlignment="Center"/>

            <!-- 可手 key 的 3 位 HEX -->
            <TextBox Width="60"
                     Text="{Binding BWLoopVM.FCNT2_Text, UpdateSourceTrigger=PropertyChanged}"
                     VerticalAlignment="Center"
                     TextAlignment="Center"
                     FontSize="16"
                     Margin="0,0,8,0"/>

            <!-- Up / Down -->
            <StackPanel Orientation="Vertical">
                <Button Content="▲"
                        Width="24" Height="16"
                        Margin="0,0,0,2"
                        Command="{Binding BWLoopVM.IncreaseFCNT2Command}"/>

                <Button Content="▼"
                        Width="24" Height="16"
                        Command="{Binding BWLoopVM.DecreaseFCNT2Command}"/>
            </StackPanel>
        </StackPanel>
    </StackPanel>
</GroupBox>