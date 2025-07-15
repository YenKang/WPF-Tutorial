<Grid Height="400" Width="120">
    <!-- Track 背景 -->
    <Border Width="30" Background="#333" CornerRadius="15" HorizontalAlignment="Center"/>

    <!-- 進度條 -->
    <Rectangle Width="30"
               Fill="DodgerBlue"
               HorizontalAlignment="Center"
               VerticalAlignment="Bottom"
               Height="{Binding VolumeValue, Converter={StaticResource ValueToHeightConverter}}"/>

    <!-- 自訂 Thumb -->
    <Ellipse Width="60" Height="60"
             Fill="LightSkyBlue"
             Stroke="White"
             StrokeThickness="2"
             HorizontalAlignment="Center"
             Margin="0,{Binding VolumeValue, Converter={StaticResource ValueToTopMarginConverter}},0,0"/>
</Grid>