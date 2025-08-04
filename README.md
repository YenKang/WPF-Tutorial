<Grid Width="Auto" Height="Auto" Background="LightGray" Margin="20">
    <Grid.RowDefinitions>
        <RowDefinition Height="250"/> <!-- 上方綠色區塊 -->
        <RowDefinition Height="300"/> <!-- 下方藍色區 + 兩個圓形 -->
    </Grid.RowDefinitions>

    <!-- 上半區域：綠色長方形 -->
    <Border Grid.Row="0" Background="LightGreen" Margin="20" CornerRadius="20" />

    <!-- 下半區域：三欄配置 -->
    <Grid Grid.Row="1" Margin="20">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="200"/>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="200"/>
        </Grid.ColumnDefinitions>

        <!-- 左旋鈕 -->
        <Ellipse Grid.Column="0" Fill="DarkGray" Width="120" Height="120" HorizontalAlignment="Center" VerticalAlignment="Center"/>

        <!-- 中間藍色長方形 -->
        <Border Grid.Column="1" Background="DodgerBlue" Height="80" CornerRadius="12"
                HorizontalAlignment="Stretch" VerticalAlignment="Center" Margin="20,0"/>

        <!-- 右旋鈕 -->
        <Ellipse Grid.Column="2" Fill="DarkGray" Width="120" Height="120" HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </Grid>
</Grid>

..