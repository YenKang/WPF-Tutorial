<GroupBox Header="Flicker RGB">
    <Grid Margin="12">

        <!-- Row: Header + Page1~4 + Button -->
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/> <!-- Header -->
            <RowDefinition Height="Auto"/> <!-- Page1 -->
            <RowDefinition Height="Auto"/> <!-- Page2 -->
            <RowDefinition Height="Auto"/> <!-- Page3 -->
            <RowDefinition Height="Auto"/> <!-- Page4 -->
            <RowDefinition Height="Auto"/> <!-- Button -->
        </Grid.RowDefinitions>

        <!-- Column: PageLabel + R + G + B -->
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="80"/>   <!-- Page Label -->
            <ColumnDefinition Width="80"/>   <!-- R -->
            <ColumnDefinition Width="80"/>   <!-- G -->
            <ColumnDefinition Width="80"/>   <!-- B -->
        </Grid.ColumnDefinitions>

        <!-- ===== Header: R G B ===== -->
        <TextBlock Grid.Row="0" Grid.Column="1" Text="R"
                   HorizontalAlignment="Center" Foreground="Red"/>
        <TextBlock Grid.Row="0" Grid.Column="2" Text="G"
                   HorizontalAlignment="Center" Foreground="Green"/>
        <TextBlock Grid.Row="0" Grid.Column="3" Text="B"
                   HorizontalAlignment="Center" Foreground="Blue"/>

        <!-- ===== Page 1 ===== -->
        <TextBlock Grid.Row="1" Grid.Column="0" Text="Page 1"
                   VerticalAlignment="Center" FontWeight="Bold"
                   Margin="0,4,0,4"/>

        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="1"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage1, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="2"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage1, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="3"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage1, Mode=TwoWay}"/>

        <!-- ===== Page 2 ===== -->
        <TextBlock Grid.Row="2" Grid.Column="0" Text="Page 2"
                   VerticalAlignment="Center" FontWeight="Bold"
                   Margin="0,4,0,4"/>

        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="1"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage2, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="2"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage2, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="3"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage2, Mode=TwoWay}"/>

        <!-- ===== Page 3 ===== -->
        <TextBlock Grid.Row="3" Grid.Column="0" Text="Page 3"
                   VerticalAlignment="Center" FontWeight="Bold"
                   Margin="0,4,0,4"/>

        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="1"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage3, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="2"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage3, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="3"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage3, Mode=TwoWay}"/>

        <!-- ===== Page 4 ===== -->
        <TextBlock Grid.Row="4" Grid.Column="0" Text="Page 4"
                   VerticalAlignment="Center" FontWeight="Bold"
                   Margin="0,4,0,4"/>

        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="1"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage4, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="2"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage4, Mode=TwoWay}"/>

        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="3"
                            Width="60" Minimum="0" Maximum="15"
                            FormatString="X2"
                            Margin="0,4,0,4"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage4, Mode=TwoWay}"/>

        <!-- ===== Set Button (右下角) ===== -->
        <Button Grid.Row="5" Grid.Column="0" Grid.ColumnSpan="4"
                Content="Set Flicker RGB"
                Width="140" Height="28"
                Margin="0,8,0,0"
                HorizontalAlignment="Right"
                Command="{Binding FlickerRGBVM.ApplyCommand}"/>

    </Grid>
</GroupBox>
