<GroupBox Header="Black White Loop"
          Margin="0,8,0,0"
          Visibility="{Binding IsBWLoopConfigVisible,
                               Converter={utilityConv:BoolToVisibilityConverter}}">

    <!-- 這裡 DataContext 綁到 BWLoopVM -->
    <StackPanel Margin="10"
                DataContext="{Binding BWLoopVM}">

        <!-- 上面一列：FCNT2 + TextBox + Up/Down -->
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="FCNT2:"
                       Width="80"
                       VerticalAlignment="Center"/>

            <!-- 可手 key 三位 HEX -->
            <TextBox Width="60"
                     Text="{Binding FCNT2_Text, UpdateSourceTrigger=PropertyChanged}"
                     VerticalAlignment="Center"
                     TextAlignment="Center"
                     FontSize="16"
                     Margin="0,0,8,0"/>

            <!-- Up / Down -->
            <StackPanel Orientation="Vertical">
                <Button Content="▲"
                        Width="24" Height="16"
                        Margin="0,0,0,2"
                        Command="{Binding IncreaseFCNT2Command}"/>

                <Button Content="▼"
                        Width="24" Height="16"
                        Command="{Binding DecreaseFCNT2Command}"/>
            </StackPanel>
        </StackPanel>

        <!-- 下面這顆就是你原本的 Set button -->
        <Button Content="Set"
                Width="80"
                Margin="0,8,0,0"
                HorizontalAlignment="Right"
                Command="{Binding ApplyCommand}"/>
        <!-- 注意：因為 DataContext 已經是 BWLoopVM，所以這裡直接綁 ApplyCommand 就好 -->
    </StackPanel>
</GroupBox>