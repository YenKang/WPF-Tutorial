<Canvas Width="2560" Height="1440" Background="Black">
    <!-- Zoom In Button -->
    <Button x:Name="ZoomInBtn"
            Content="Zoom In"
            Width="{Binding ZoomInWidth}"
            Height="{Binding ZoomInHeight}"
            Canvas.Left="{Binding ZoomInX}"
            Canvas.Top="{Binding ZoomInY}"
            FontSize="40"
            CommandParameter="+1"
            Command="{Binding ClickToSetZoomLevelCommand}" />

    <!-- 校正框：可選 -->
    <Rectangle Width="{Binding ZoomInWidth}"
               Height="{Binding ZoomInHeight}"
               Canvas.Left="{Binding ZoomInX}"
               Canvas.Top="{Binding ZoomInY}"
               Stroke="Lime"
               StrokeThickness="2"
               Fill="Transparent" />
</Canvas>