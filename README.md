<Canvas>
    <!-- 切換歌曲 icon -->
    <Image Source="{Binding LeftKnobInnerImage}"
           Width="220"
           Height="220"
           Visibility="{Binding IsLeftKnobInnerVisible, Converter={StaticResource BoolToVis}}"
           Canvas.Left="313"
           Canvas.Top="1020"/>
</Canvas>