<StackPanel Orientation="Horizontal" VerticalAlignment="Center" Margin="20">
    <!-- 左側文字 -->
    <TextBlock Text="Enable BIST Mode"
               VerticalAlignment="Center"
               Foreground="White"
               FontSize="16"
               Margin="0,0,12,0"/>

    <!-- 右側開關 -->
    <ToggleButton x:Name="BistSwitch"
                  Width="60"
                  Height="28"
                  IsChecked="{Binding IsBistEnabled}"
                  Cursor="Hand">
        <ToggleButton.Template>
            <ControlTemplate TargetType="ToggleButton">
                <Grid>
                    <!-- 背景條 -->
                    <Border x:Name="SwitchBackground"
                            CornerRadius="14"
                            Background="#555"
                            Width="60" Height="28"/>

                    <!-- 滑塊圓形 -->
                    <Ellipse x:Name="Knob"
                             Width="22" Height="22"
                             Fill="White"
                             Margin="3,3,0,3"
                             HorizontalAlignment="Left"/>

                    <!-- 文字顯示 On / Off -->
                    <TextBlock x:Name="SwitchLabel"
                               Text="Off"
                               Foreground="White"
                               HorizontalAlignment="Center"
                               VerticalAlignment="Center"
                               FontWeight="Bold"
                               FontSize="12"
                               Opacity="0.7"/>
                </Grid>

                <!-- 狀態觸發 -->
                <ControlTemplate.Triggers>
                    <!-- ON 狀態 -->
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