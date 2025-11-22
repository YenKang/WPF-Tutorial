<GroupBox Header="Flicker RGB">
    <GroupBox.DataContext>
        <Binding Path="FlickerRGBVM"/>
    </GroupBox.DataContext>

    <StackPanel Margin="12">

        <!-- Page1 -->
        <TextBlock Text="Page 1" FontWeight="Bold" Margin="0,8,0,2"/>
        <StackPanel Orientation="Horizontal" Margin="0,0,0,6" Spacing="20">

            <!-- R -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding RPage1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="R"
                           HorizontalAlignment="Center"
                           Foreground="Red"/>
            </StackPanel>

            <!-- G -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding GPage1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="G"
                           HorizontalAlignment="Center"
                           Foreground="Green"/>
            </StackPanel>

            <!-- B -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding BPage1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="B"
                           HorizontalAlignment="Center"
                           Foreground="Blue"/>
            </StackPanel>

        </StackPanel>


        <!-- Page2 -->
        <TextBlock Text="Page 2" FontWeight="Bold" Margin="0,8,0,2"/>
        <StackPanel Orientation="Horizontal" Margin="0,0,0,6" Spacing="20">

            <!-- R -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding RPage2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="R" HorizontalAlignment="Center" Foreground="Red"/>
            </StackPanel>

            <!-- G -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding GPage2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="G" HorizontalAlignment="Center" Foreground="Green"/>
            </StackPanel>

            <!-- B -->
            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding BPage2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                <TextBlock Text="B" HorizontalAlignment="Center" Foreground="Blue"/>
            </StackPanel>

        </StackPanel>


        <!-- Page3 -->
        <TextBlock Text="Page 3" FontWeight="Bold" Margin="0,8,0,2"/>
        <StackPanel Orientation="Horizontal" Margin="0,0,0,6" Spacing="20">

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding RPage3, Mode=TwoWay}"/>
                <TextBlock Text="R" HorizontalAlignment="Center" Foreground="Red"/>
            </StackPanel>

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding GPage3, Mode=TwoWay}"/>
                <TextBlock Text="G" HorizontalAlignment="Center" Foreground="Green"/>
            </StackPanel>

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding BPage3, Mode=TwoWay}"/>
                <TextBlock Text="B" HorizontalAlignment="Center" Foreground="Blue"/>
            </StackPanel>

        </StackPanel>


        <!-- Page4 -->
        <TextBlock Text="Page 4" FontWeight="Bold" Margin="0,8,0,2"/>
        <StackPanel Orientation="Horizontal" Margin="0,0,0,6" Spacing="20">

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding RPage4, Mode=TwoWay}"/>
                <TextBlock Text="R" HorizontalAlignment="Center" Foreground="Red"/>
            </StackPanel>

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding GPage4, Mode=TwoWay}"/>
                <TextBlock Text="G" HorizontalAlignment="Center" Foreground="Green"/>
            </StackPanel>

            <StackPanel>
                <xctk:IntegerUpDown Width="60"
                                    Minimum="0"
                                    Maximum="15"
                                    Value="{Binding BPage4, Mode=TwoWay}"/>
                <TextBlock Text="B" HorizontalAlignment="Center" Foreground="Blue"/>
            </StackPanel>

        </StackPanel>

    </StackPanel>
</GroupBox>
