<!-- 3-8) Chess BoardControl -->
<GroupBox Header="Chessboard"
          Visibility="{Binding IsChessBoardConfigVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,8,0,0">
    <StackPanel>

        <!-- Resolution：HRes / VRes 也改成 UpDown，可手 key -->
        <StackPanel Orientation="Horizontal" Margin="0,4,0,0">
            <TextBlock Text="Resolution:" Width="100" VerticalAlignment="Center" />

            <xctk:IntegerUpDown
                Width="80" Height="24" Margin="0,0,4,0"
                Minimum="720"
                Maximum="6720"
                Value="{Binding ChessBoardVM.HRes,
                                Mode=TwoWay,
                                UpdateSourceTrigger=PropertyChanged}" />

            <TextBlock Text="×" Margin="0,0,4,0" VerticalAlignment="Center" />

            <xctk:IntegerUpDown
                Width="80" Height="24"
                Minimum="136"
                Maximum="2560"
                Value="{Binding ChessBoardVM.VRes,
                                Mode=TwoWay,
                                UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- M / N：已經改成 UpDown -->
        <StackPanel Orientation="Horizontal" Margin="0,4,0,10">
            <TextBlock Text="M (H split):"
                       Width="100"
                       VerticalAlignment="Center" />

            <xctk:IntegerUpDown
                Width="60" Height="30"
                Minimum="2"
                Maximum="40"
                Value="{Binding ChessBoardVM.M,
                                Mode=TwoWay,
                                UpdateSourceTrigger=PropertyChanged}" />

            <TextBlock Text="N (V split):"
                       Width="100"
                       Margin="24,0,0,0"
                       VerticalAlignment="Center" />

            <xctk:IntegerUpDown
                Width="60" Height="30"
                Minimum="2"
                Maximum="40"
                Value="{Binding ChessBoardVM.N,
                                Mode=TwoWay,
                                UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- Set -->
        <Button Content="Set"
                Width="120"
                Command="{Binding ChessBoardVM.ApplyCommand}" />
    </StackPanel>
</GroupBox>