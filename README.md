<Canvas Width="2560" Height="1440" Background="Transparent">

    <Ellipse x:Name="LeftKnobOuter"
             Width="371" Height="371"
             Fill="Transparent"
             Stroke="Red"
             StrokeThickness="4"
             IsHitTestVisible="True"
             MouseEnter="LeftKnobOuter_MouseEnter"
             MouseLeave="LeftKnobOuter_MouseLeave" />

    <Ellipse x:Name="LeftKnobGlow"
             Width="400" Height="400"
             Fill="Cyan"
             Opacity="0"
             StrokeThickness="0"
             IsHitTestVisible="False"/>
</Canvas>