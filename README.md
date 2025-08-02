ˋˋˋxml
<Grid Background="Black">

    <!-- 定義三列：Top / Center / Bottom -->
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>   <!-- Logo / Page Title 區 -->
        <RowDefinition Height="*"/>      <!-- 中央主視覺 -->
        <RowDefinition Height="Auto"/>   <!-- 底部 Knob + 按鈕列 -->
    </Grid.RowDefinitions>

    <!-- 頂部：Logo + Page 標題 -->
    <DockPanel Grid.Row="0" Margin="40,20">
        <Image Source="/Assets/Logo.png" Height="50" DockPanel.Dock="Left"/>
        <TextBlock Text="MUSIC" Foreground="White" FontSize="32" VerticalAlignment="Center" Margin="20,0,0,0"/>
    </DockPanel>

    <!-- 中央主區塊：專輯封面 or 主視覺 -->
    <Grid Grid.Row="1" HorizontalAlignment="Center" VerticalAlignment="Center">
        <Border Width="600" Height="600" Background="Gray" CornerRadius="20">
            <Image Source="/Assets/AlbumCover.png" Stretch="UniformToFill"/>
        </Border>
    </Grid>

    <!-- 底部：左右 Knob + 中間按鈕列 -->
    <Grid Grid.Row="2" Margin="20" VerticalAlignment="Bottom">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="2*"/>  <!-- 左旋鈕 -->
            <ColumnDefinition Width="3*"/>  <!-- 中間按鈕列 -->
            <ColumnDefinition Width="2*"/>  <!-- 右旋鈕 -->
        </Grid.ColumnDefinitions>

        <!-- 左 Knob -->
        <StackPanel Grid.Column="0" HorizontalAlignment="Center" VerticalAlignment="Center">
            <Ellipse Width="200" Height="200" Fill="DarkGray"/>
            <TextBlock Text="Driver Knob" Foreground="White" HorizontalAlignment="Center" Margin="0,10,0,0"/>
        </StackPanel>

        <!-- 中間按鈕列 -->
        <StackPanel Grid.Column="1" Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Spacing="20">
            <Button Content="Home" Width="100" Height="50"/>
            <Button Content="Music" Width="100" Height="50"/>
            <Button Content="Climate" Width="100" Height="50"/>
        </StackPanel>

        <!-- 右 Knob -->
        <StackPanel Grid.Column="2" HorizontalAlignment="Center" VerticalAlignment="Center">
            <Ellipse Width="200" Height="200" Fill="DarkGray"/>
            <TextBlock Text="Passenger Knob" Foreground="White" HorizontalAlignment="Center" Margin="0,10,0,0"/>
        </StackPanel>

    </Grid>
</Grid>
ˋˋˋ
