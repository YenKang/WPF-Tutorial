<Window x:Class="MusicPageDemo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Music Page" Height="800" Width="1400" Background="#FF0D0D0D">

    <Grid>
        <!-- 中央內容 -->
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Spacing="20">
            <!-- 專輯封面 -->
            <Image Source="Assets/MichaelJackson.png" Width="250" Height="250" />

            <!-- 歌曲資訊 -->
            <TextBlock Text="Beat It" Foreground="White" FontSize="30" FontWeight="Bold" HorizontalAlignment="Center"/>
            <TextBlock Text="Michael Jackson" Foreground="Gray" FontSize="18" HorizontalAlignment="Center"/>

            <!-- 進度條 -->
            <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="0,10,0,0">
                <TextBlock Text="1:04" Foreground="White" VerticalAlignment="Center" Margin="0,0,10,0"/>
                <ProgressBar Width="400" Height="6" Value="30" Maximum="100" Foreground="DodgerBlue" Background="#333"/>
                <TextBlock Text="3:52" Foreground="White" VerticalAlignment="Center" Margin="10,0,0,0"/>
            </StackPanel>

            <!-- 播放控制按鈕 -->
            <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Spacing="30" Margin="0,10,0,0">
                <Button Content="⏮" Width="60" Height="60" FontSize="24" Background="Transparent" Foreground="White" BorderBrush="Transparent"/>
                <Button Content="▶" Width="60" Height="60" FontSize="24" Background="Transparent" Foreground="White" BorderBrush="Transparent"/>
                <Button Content="⏭" Width="60" Height="60" FontSize="24" Background="Transparent" Foreground="White" BorderBrush="Transparent"/>
            </StackPanel>
        </StackPanel>

        <!-- 右側音量條 -->
        <StackPanel HorizontalAlignment="Right" VerticalAlignment="Center" Margin="0,0,80,0">
            <Slider Orientation="Vertical" Minimum="0" Maximum="100" Value="50" Width="40" Height="300"
                    Foreground="DodgerBlue" Background="#333" 
                    Style="{StaticResource {x:Static ToolBar.VerticalSliderStyleKey}}"/>
            <TextBlock Text="🔊" Foreground="White" FontSize="24" HorizontalAlignment="Center" Margin="0,10,0,0"/>
        </StackPanel>

    </Grid>
</Window>