<GroupBox Header="{Binding AutoRunVM.Title}"
          Margin="0,12,0,12">

    <GroupBox.DataContext>
        <Binding Path="AutoRunVM"/>
    </GroupBox.DataContext>

    <StackPanel Margin="12">

        <!-- Pattern Total -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,16">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Width="120"/>
            <xctk:IntegerUpDown Width="80"
                                Minimum="1"
                                Maximum="30"
                                Value="{Binding Total, Mode=TwoWay}"/>
        </StackPanel>

        <!-- 22 個 Pattern Order -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,4,0,4">

                        <!-- 顯示序號（你 VM 用 DisplayNo） -->
                        <TextBlock Text="{Binding DisplayNo}"
                                   VerticalAlignment="Center"
                                   Width="40"/>

                        <!-- 原本 ComboBox → 改 UpDown；綁 SelectedIndex -->
                        <xctk:IntegerUpDown Width="80"
                                            Minimum="0"
                                            Maximum="255"
                                            Value="{Binding SelectedIndex,
                                                            Mode=TwoWay,
                                                            UpdateSourceTrigger=PropertyChanged}"/>

                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <!-- FCNT1 -->
        <StackPanel Orientation="Horizontal" Margin="0,16,0,8">
            <TextBlock Text="FCNT1"
                       VerticalAlignment="Center"
                       Width="120"/>
            <xctk:IntegerUpDown Width="100"
                                Minimum="0"
                                Maximum="4095"
                                FormatString="X3"
                                Value="{Binding Fcnt1, Mode=TwoWay}"/>
        </StackPanel>

        <!-- FCNT2 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,16">
            <TextBlock Text="FCNT2"
                       VerticalAlignment="Center"
                       Width="120"/>
            <xctk:IntegerUpDown Width="100"
                                Minimum="0"
                                Maximum="4095"
                                FormatString="X3"
                                Value="{Binding Fcnt2, Mode=TwoWay}"/>
        </StackPanel>

        <!-- Apply -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding ApplyCommand}"/>

    </StackPanel>
</GroupBox>
