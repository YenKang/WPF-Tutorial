<Slider Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeValue, Mode=TwoWay}"
        Width="120"
        Height="400"
        HorizontalAlignment="Center">

    <Slider.Template>
        <ControlTemplate TargetType="Slider">
            <Grid>
                <!-- 背景 -->
                <Border Width="30" Background="#333" CornerRadius="15"/>

                <!-- Track -->
                <Track x:Name="PART_Track"
                       IsDirectionReversed="False" <!-- ✅ 這裡改成 False -->
                       Minimum="{TemplateBinding Minimum}"
                       Maximum="{TemplateBinding Maximum}"
                       Value="{TemplateBinding Value}"
                       Width="30">
                    
                    <Track.DecreaseRepeatButton>
                        <RepeatButton Command="Slider.DecreaseLarge" Background="DodgerBlue" />
                    </Track.DecreaseRepeatButton>

                    <Track.Thumb>
                        <Thumb Width="60" Height="60">
                            <Thumb.Template>
                                <ControlTemplate TargetType="Thumb">
                                    <Ellipse Fill="LightSkyBlue"
                                             Stroke="White"
                                             StrokeThickness="2"/>
                                </ControlTemplate>
                            </Thumb.Template>
                        </Thumb>
                    </Track.Thumb>

                    <Track.IncreaseRepeatButton>
                        <RepeatButton Command="Slider.IncreaseLarge" Background="Transparent" />
                    </Track.IncreaseRepeatButton>
                </Track>
            </Grid>
        </ControlTemplate>
    </Slider.Template>
</Slider>