<StackPanel Orientation="Vertical" HorizontalAlignment="Center" VerticalAlignment="Center">
    <TextBlock Text="Gray Level" FontSize="16" FontWeight="Bold" Margin="0,0,0,10"/>

    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="0,0,0,10">
        <TextBox Width="80"
                 Text="{Binding GrayLevel, UpdateSourceTrigger=PropertyChanged}"
                 MaxLength="3"
                 HorizontalContentAlignment="Center"
                 VerticalContentAlignment="Center"/>
        <StackPanel Orientation="Vertical" Margin="5,0,0,0">
            <Button Content="▲" Command="{Binding IncreaseCommand}" Width="25" Height="20"/>
            <Button Content="▼" Command="{Binding DecreaseCommand}" Width="25" Height="20"/>
        </StackPanel>
    </StackPanel>

    <Button Content="Set"
            Width="100"
            Command="{Binding SetCommand}"/>
</StackPanel>