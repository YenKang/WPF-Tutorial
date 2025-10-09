<StackPanel Margin="20">
    <!-- 標題 + 開關 -->
    <StackPanel Orientation="Horizontal" VerticalAlignment="Center">
        <TextBlock Text="Enable BIST Mode"
                   VerticalAlignment="Center"
                   Foreground="White"
                   FontSize="16"
                   Margin="0,0,12,0"/>

        <ToggleButton Width="60" Height="28"
                      Cursor="Hand"
                      IsChecked="{Binding IsBistEnabled, Mode=TwoWay}">
            <ToggleButton.Template>
                <ControlTemplate TargetType="ToggleButton">
                    <Grid>
                        <Border x:Name="SwitchBackground"
                                CornerRadius="14"
                                Background="#555"
                                Width="60" Height="28"/>
                        <Ellipse x:Name="Knob"
                                 Width="22" Height="22"
                                 Fill="White"
                                 Margin="3,3,0,3"
                                 HorizontalAlignment="Left"/>
                        <TextBlock x:Name="SwitchLabel"
                                   Text="Off"
                                   Foreground="White"
                                   HorizontalAlignment="Center"
                                   VerticalAlignment="Center"
                                   FontWeight="Bold"
                                   FontSize="12"
                                   Opacity="0.7"/>
                    </Grid>
                    <ControlTemplate.Triggers>
                        <Trigger Property="IsChecked" Value="True">
                            <Setter TargetName="SwitchBackground" Property="Background" Value="#2196F3"/>
                            <Setter TargetName="Knob" Property="HorizontalAlignment" Value="Right"/>
                            <Setter TargetName="SwitchLabel" Property="Text" Value="On"/>
                            <Setter TargetName="SwitchLabel" Property="Opacity" Value="1"/>
                        </Trigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </ToggleButton.Template>
        </ToggleButton>
    </StackPanel>

    <!-- 下方 Set 按鈕 -->
    <Button Content="Set"
            Width="80"
            Margin="0,8,0,0"
            Command="{Binding EnableBistModeCommand}"/>
</StackPanel>