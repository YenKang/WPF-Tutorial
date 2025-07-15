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

                <!-- 進度條，獨立用 Rectangle 表示 -->
                <Rectangle Width="30"
                           Fill="DodgerBlue"
                           RadiusX="15"
                           RadiusY="15"
                           VerticalAlignment="Bottom">
                    <Rectangle.Height>
                        <MultiBinding Converter="{StaticResource ValueToHeightConverter}">
                            <Binding RelativeSource="{RelativeSource TemplatedParent}" Path="Value"/>
                            <Binding RelativeSource="{RelativeSource TemplatedParent}" Path="Maximum"/>
                            <Binding RelativeSource="{RelativeSource TemplatedParent}" Path="Height"/>
                        </MultiBinding>
                    </Rectangle.Height>
                </Rectangle>

                <!-- Track（純空間控制，不再顯示顏色） -->
                <Track x:Name="PART_Track"
                       Minimum="{TemplateBinding Minimum}"
                       Maximum="{TemplateBinding Maximum}"
                       Value="{TemplateBinding Value}"
                       IsDirectionReversed="False"
                       Width="30">
                    
                    <Track.DecreaseRepeatButton>
                        <RepeatButton Background="Transparent" IsEnabled="False"/>
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
                        <RepeatButton Background="Transparent" IsEnabled="False"/>
                    </Track.IncreaseRepeatButton>
                </Track>
            </Grid>
        </ControlTemplate>
    </Slider.Template>
</Slider>


```csharp
public class ValueToHeightConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        double value = (double)values[0];
        double max = (double)values[1];
        double totalHeight = (double)values[2];

        if (max == 0) return 0;

        double percent = value / max;
        return percent * totalHeight;
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```
