<Button Width="180" Height="180" Margin="50" Command="{Binding NavigateToMusicCommand}" Background="Transparent" BorderThickness="0">
    <Button.Template>
        <ControlTemplate TargetType="Button">
            <StackPanel>
                <Grid Width="150" Height="150">
                    <!-- 圓形背景 -->
                    <Ellipse x:Name="CircleBackground" Fill="Transparent" Stroke="Cyan" StrokeThickness="2"/>

                    <!-- 中央 Icon -->
                    <Image x:Name="IconImage"
                           Source="/Assets/Icons/music_icon.png"
                           Width="80" Height="80"
                           Stretch="Uniform"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Center"
                           Opacity="0.8"/>
                </Grid>

                <!-- 文字 -->
                <TextBlock x:Name="LabelText"
                           Text="Music"
                           Foreground="White"
                           FontSize="22"
                           HorizontalAlignment="Center"
                           Margin="0,10,0,0"
                           Opacity="0.8"/>
            </StackPanel>

            <!-- Hover 特效 -->
            <ControlTemplate.Triggers>
                <Trigger Property="IsMouseOver" Value="True">
                    <Setter TargetName="CircleBackground" Property="Fill" Value="#4400FFFF"/> <!-- 半透明亮藍 -->
                    <Setter TargetName="CircleBackground" Property="Stroke" Value="#00FFFF"/>
                    <Setter TargetName="IconImage" Property="Opacity" Value="1"/>
                    <Setter TargetName="LabelText" Property="Foreground" Value="#00FFFF"/>
                    <Setter TargetName="LabelText" Property="Opacity" Value="1"/>
                </Trigger>
            </ControlTemplate.Triggers>
        </ControlTemplate>
    </Button.Template>
</Button>