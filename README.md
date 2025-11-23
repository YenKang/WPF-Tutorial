<GroupBox Header="{Binding AutoRunVM.Title}"
          Margin="0,12,0,12">

    <!-- 使用外層 AutoRunVM -->
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

        <!-- 固定 22 筆 Pattern Order（每筆一個 UpDown） -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,4,0,4">

                        <!-- 顯示序號 1~22 -->
                        <TextBlock Text="{Binding Index}"
                                   VerticalAlignment="Center"
                                   Width="40"/>

                        <!-- UpDown (raw value) -->
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
                                Value="{Binding Fcnt1, Mode=TwoWay}"
                                FormatString="X3"/>
        </StackPanel>

        <!-- FCNT2 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,16">
            <TextBlock Text="FCNT2"
                       VerticalAlignment="Center"
                       Width="120"/>
            <xctk:IntegerUpDown Width="100"
                                Minimum="0"
                                Maximum="4095"
                                Value="{Binding Fcnt2, Mode=TwoWay}"
                                FormatString="X3"/>
        </StackPanel>

        <!-- Apply -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding ApplyCommand}"/>

    </StackPanel>
</GroupBox>
