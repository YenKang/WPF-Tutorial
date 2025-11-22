<GroupBox Header="Flicker RGB" Margin="0,12,0,12">
    <Grid Margin="12">
        <!-- 5 列：標題 + Page1~4 -->
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/> <!-- Header: R G B -->
            <RowDefinition Height="Auto"/> <!-- Page1 -->
            <RowDefinition Height="Auto"/> <!-- Page2 -->
            <RowDefinition Height="Auto"/> <!-- Page3 -->
            <RowDefinition Height="Auto"/> <!-- Page4 -->
        </Grid.RowDefinitions>

        <!-- 4 欄：Page 標籤 + R/G/B -->
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="80"/>  <!-- Page label -->
            <ColumnDefinition Width="80"/>  <!-- R -->
            <ColumnDefinition Width="80"/>  <!-- G -->
            <ColumnDefinition Width="80"/>  <!-- B -->
        </Grid.ColumnDefinitions>

        <!-- ===== Header Row: R G B ===== -->
        <TextBlock Grid.Row="0" Grid.Column="1"
                   Text="R"
                   HorizontalAlignment="Center"
                   Foreground="Red"/>
        <TextBlock Grid.Row="0" Grid.Column="2"
                   Text="G"
                   HorizontalAlignment="Center"
                   Foreground="Green"/>
        <TextBlock Grid.Row="0" Grid.Column="3"
                   Text="B"
                   HorizontalAlignment="Center"
                   Foreground="Blue"/>

        <!-- ===== Page 1 ===== -->
        <TextBlock Grid.Row="1" Grid.Column="0"
                   Text="Page 1"
                   VerticalAlignment="Center"
                   FontWeight="Bold"/>

        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="1"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage1, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="2"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage1, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="3"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage1, Mode=TwoWay}"/>

        <!-- ===== Page 2 ===== -->
        <TextBlock Grid.Row="2" Grid.Column="0"
                   Text="Page 2"
                   VerticalAlignment="Center"
                   FontWeight="Bold"/>

        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="1"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage2, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="2"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage2, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="3"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage2, Mode=TwoWay}"/>

        <!-- ===== Page 3 ===== -->
        <TextBlock Grid.Row="3" Grid.Column="0"
                   Text="Page 3"
                   VerticalAlignment="Center"
                   FontWeight="Bold"/>

        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="1"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage3, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="2"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage3, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="3"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage3, Mode=TwoWay}"/>

        <!-- ===== Page 4 ===== -->
        <TextBlock Grid.Row="4" Grid.Column="0"
                   Text="Page 4"
                   VerticalAlignment="Center"
                   FontWeight="Bold"/>

        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="1"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.RPage4, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="2"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.GPage4, Mode=TwoWay}"/>
        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="3"
                            Width="60"
                            Minimum="0"
                            Maximum="15"
                            HorizontalContentAlignment="Center"
                            Value="{Binding FlickerRGBVM.BPage4, Mode=TwoWay}"/>

    </Grid>
</GroupBox>
