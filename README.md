<!-- 3-8) Chess BoardControl -->
<GroupBox Header="Chessboard"
          Visibility="{Binding IsChessBoardConfigVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,8,0,0">
    <StackPanel>

        <!-- Resolution -->
        <StackPanel Orientation="Horizontal" Margin="0,4,0,0">
            <TextBlock Text="Resolution:" Width="100" />
            <TextBlock Width="80"
                       Text="{Binding ChessBoardVM.HRes, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
            <TextBlock Text=" × " Margin="0,0,4,0" />
            <TextBlock Width="80"
                       Text="{Binding ChessBoardVM.VRes, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- M / N：改用 IntegerUpDown -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,10">
            <TextBlock Text="M (H split):" Width="100" VerticalAlignment="Center" />

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