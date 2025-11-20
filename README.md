<GroupBox
    Header="{Binding AutoRunVM.Title}"
    Visibility="{Binding IsAutoRunConfigVisible,
                         Converter={utilityConv:BoolToVisibilityConverter}}"
    Margin="0,12,0,0">

    <StackPanel x:Name="AutoRunPanel"
                Margin="12"
                DataContext="{Binding AutoRunVM}">

        <!-- Pattern Total -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Margin="0,0,8,0" />

            <xctk:IntegerUpDown Width="96"
                                Minimum="1"
                                Maximum="22"
                                Value="{Binding Total,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- Pattern Orders -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,2,0,2">

                        <TextBlock Width="120"
                                   VerticalAlignment="Center"
                                   Text="{Binding DisplayNo,
                                          StringFormat=Pattern Order: {0}}"/>

                        <ComboBox Width="96"
                                  ItemsSource="{Binding DataContext.PatternNumberOptions,
                                                        ElementName=AutoRunPanel}"
                                  SelectedItem="{Binding SelectedPatternNumber,
                                                         Mode=TwoWay,
                                                         UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                Margin="0,12,0,0"
                Command="{Binding ApplyCommand}" />

    </StackPanel>
</GroupBox>
