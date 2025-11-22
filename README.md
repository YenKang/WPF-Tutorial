<GroupBox Header="{Binding GrayColorVM.Title}"
          Margin="0,12,0,12">

    <StackPanel Margin="12">

        <!-- R -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <CheckBox Content="R"
                      Width="50"
                      VerticalAlignment="Center"
                      IsChecked="{Binding GrayColorVM.IsREnabled,
                                          Mode=TwoWay,
                                          UpdateSourceTrigger=PropertyChanged}"/>

            <Border Width="24" Height="24" Background="Red"
                    BorderBrush="Black" BorderThickness="1"
                    Margin="6,0,0,0"/>
        </StackPanel>

        <!-- G -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <CheckBox Content="G"
                      Width="50"
                      VerticalAlignment="Center"
                      IsChecked="{Binding GrayColorVM.IsGEnabled,
                                          Mode=TwoWay,
                                          UpdateSourceTrigger=PropertyChanged}"/>

            <Border Width="24" Height="24" Background="Lime"
                    BorderBrush="Black" BorderThickness="1"
                    Margin="6,0,0,0"/>
        </StackPanel>

        <!-- B -->
        <StackPanel Orientation="Horizontal">
            <CheckBox Content="B"
                      Width="50"
                      VerticalAlignment="Center"
                      IsChecked="{Binding GrayColorVM.IsBEnabled,
                                          Mode=TwoWay,
                                          UpdateSourceTrigger=PropertyChanged}"/>

            <Border Width="24" Height="24" Background="Blue"
                    BorderBrush="Black" BorderThickness="1"
                    Margin="6,0,0,0"/>
        </StackPanel>

        <Button Content="Set GrayColor"
                Width="140"
                Height="32"
                Margin="0,12,0,0"
                Command="{Binding GrayColorVM.ApplyCommand}"/>

    </StackPanel>
</GroupBox>
