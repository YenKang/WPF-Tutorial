<Slider Orientation="Vertical"
        Minimum="0"
        Maximum="10"
        Value="{Binding VolumeValue, Mode=TwoWay}"
        Width="100"
        Height="400"
        HorizontalAlignment="Center">

    <Slider.Template>
        <ControlTemplate TargetType="Slider">
            <Grid>
                <!-- Track -->
                <Track x:Name="PART_Track"
                       IsDirectionReversed="true"
                       Minimum="{TemplateBinding Minimum}"
                       Maximum="{TemplateBinding Maximum}"
                       Value="{TemplateBinding Value}">
                    
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
                </Track>
            </Grid>
        </ControlTemplate>
    </Slider.Template>
</Slider>