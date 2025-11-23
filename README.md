<GroupBox Header="Auto Run Config"
          Margin="0,12,0,12"
          Visibility="{Binding IsAutoRunConfigVisible}">

    <StackPanel Margin="12">

        <!-- Pattern Total -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,16">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Width="120"/>
            <xctk:IntegerUpDown Width="80"
                                Minimum="1"
                                Maximum="30"
                                Value="{Binding AutoRunVM.Total,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- Pattern Order 1~N -->
        <ItemsControl ItemsSource="{Binding AutoRunVM.Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,4,0,4">

                        <!-- 顯示序號 -->
                        <TextBlock Text="{Binding DisplayNo}"
                                   VerticalAlignment="Center"
                                   Width="40"/>

                        <!-- UpDown 調整 SelectedIndex -->
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
                                Value="{Binding AutoRunVM.Fcnt1,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
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
                                Value="{Binding AutoRunVM.Fcnt2,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}"/>
        </StackPanel>

        <!-- Apply -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding AutoRunVM.ApplyCommand}"/>

    </StackPanel>
</GroupBox>
