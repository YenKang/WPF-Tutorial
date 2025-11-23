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

        <!-- Pattern Orders (UpDown 格式) -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,4,0,4">

                        <!-- 序號：來自 OrderSlot.Index -->
                        <TextBlock Text="{Binding Index}"
                                   Width="40"
                                   VerticalAlignment="Center"/>

                        <!-- UpDown：raw 數值 -->
                        <xctk:IntegerUpDown Width="80"
                                            Minimum="0"
                                            Maximum="255"
                                            Value="{Binding Value, Mode=TwoWay}"/>

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

        <!-- Apply Button -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding ApplyCommand}"/>

    </StackPanel>
</GroupBox>
