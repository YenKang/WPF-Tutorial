## xaml

```
<GroupBox Header="Flicker RGB Control"
          Margin="0,12,0,12">

    <StackPanel Margin="12">

        <!--====================== Page 1 ======================-->
        <TextBlock Text="Page 1" FontWeight="Bold" Margin="0,0,0,6"/>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="R1" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.RPage1, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="G1" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.GPage1, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="B1" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.BPage1, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>


        <!--====================== Page 2 ======================-->
        <TextBlock Text="Page 2" FontWeight="Bold" Margin="0,8,0,6"/>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="R2" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.RPage2, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="G2" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.GPage2, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="B2" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.BPage2, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>


        <!--====================== Page 3 ======================-->
        <TextBlock Text="Page 3" FontWeight="Bold" Margin="0,8,0,6"/>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="R3" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.RPage3, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="G3" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.GPage3, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="B3" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.BPage3, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>


        <!--====================== Page 4 ======================-->
        <TextBlock Text="Page 4" FontWeight="Bold" Margin="0,8,0,6"/>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="R4" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.RPage4, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,4">
            <TextBlock Text="G4" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.GPage4, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <StackPanel Orientation="Horizontal" Margin="0,0,0,12">
            <TextBlock Text="B4" Width="40" VerticalAlignment="Center"/>
            <xctk:IntegerUpDown
                Width="70"
                Minimum="0"
                Maximum="15"
                Value="{Binding FlickerRGBVM.BPage4, Mode=TwoWay}"
                FormatString="X2" />
        </StackPanel>

        <!-- Write button -->
        <Button Content="Set Flicker RGB"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Command="{Binding FlickerRGBVM.ApplyCommand}"/>
    </StackPanel>
</GroupBox>
```
