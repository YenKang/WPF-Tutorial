<Canvas Width="2560" Height="1440" Background="Black">
    <!-- Zoom In Button -->
    <Button Content="Zoom In"
            Width="180"
            Height="100"
            Canvas.Left="200"
            Canvas.Top="1150"
            FontSize="30"
            CommandParameter="+1"
            />

    <!-- 描邊框（對齊用） -->
    <Rectangle Width="180"
               Height="100"
               Canvas.Left="200"
               Canvas.Top="1150"
               Stroke="Lime"
               StrokeThickness="2"
               Fill="Transparent"/>
</Canvas>