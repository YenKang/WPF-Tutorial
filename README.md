<Slider Orientation="Vertical" Minimum="0" Maximum="100" Value="50" Width="40" Height="300"
        Foreground="DodgerBlue" Background="#333"/><Window x:Class="MusicPageDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Music Page" Height="800" Width="1400" Background="#FF0D0D0D">
    
    <Grid>
        <!-- 左邊專輯與資訊 -->
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Spacing="20">
            <!-- 專輯封面 -->
            <Image Source="Assets/YourAlbumCover.png" Width="300" Height="300" 
                   Stretch="UniformToFill" />

            <!-- 歌名 -->
            <TextBlock Text="ceilings" Foreground="White" 
                       FontSize="32" FontWeight="Bold" HorizontalAlignment="Center"/>

            <!-- 歌手 -->
            <TextBlock Text="Lizzy McAlpine" Foreground="Gray" 
                       FontSize="20" HorizontalAlignment="Center"/>

            <!-- 音波裝飾條 (裝飾用) -->
            <Rectangle Width="300" Height="40" Fill="#444" RadiusX="10" RadiusY="10" />

            <!-- 播放控制 -->
            <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Spacing="30">
                <Button Content="⏮" Width="60" Height="60" FontSize="24" 
                        Background="Transparent" Foreground="White" BorderBrush="Transparent"/>
                <Button Content="▶" Width="80" Height="80" FontSize="28" 
                        Background="#222" Foreground="White" BorderBrush="Transparent" 
                        CornerRadius="40"/>
                <Button Content="⏭" Width="60" Height="60" FontSize="24" 
                        Background="Transparent" Foreground="White" BorderBrush="Transparent"/>
            </StackPanel>
        </StackPanel>

        <!-- 右側音量條 -->
        <StackPanel HorizontalAlignment="Right" VerticalAlignment="Center" Margin="0,0,80,0">
            <Slider Orientation="Vertical" Minimum="0" Maximum="100" Value="50" 
                    Width="40" Height="300" 
                    Foreground="DodgerBlue" Background="#333"/>

            <TextBlock Text="🔊" Foreground="White" FontSize="26" 
                       HorizontalAlignment="Center" Margin="0,10,0,0"/>
        </StackPanel>

    </Grid>
</Window>