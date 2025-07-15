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
                <Track x:Name="PART_Track"
                       Minimum="{TemplateBinding Minimum}"
                       Maximum="{TemplateBinding Maximum}"
                       Value="{TemplateBinding Value}"
                       IsDirectionReversed="False">

                    <!-- 已填滿區域 -->
                    <Track.DecreaseRepeatButton>
                        <RepeatButton Background="DodgerBlue" IsEnabled="False" Height="0"/>
                    </Track.DecreaseRepeatButton>

                    <!-- Thumb -->
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

                    <!-- 尚未填滿區域 -->
                    <Track.IncreaseRepeatButton>
                        <RepeatButton Background="#333" IsEnabled="False" Height="0"/>
                    </Track.IncreaseRepeatButton>
                </Track>
            </Grid>
        </ControlTemplate>
    </Slider.Template>
</Slider>