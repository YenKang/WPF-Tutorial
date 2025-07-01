<Ellipse x:Name="LeftKnobCenterPoint"
         Width="10" Height="10"
         Fill="Red"
         Opacity="1"
         Stroke="White"
         StrokeThickness="1"
         IsHitTestVisible="False"/>


Canvas.SetLeft(LeftKnobCenterPoint, centerX - 5); // 半徑 = 5
Canvas.SetTop(LeftKnobCenterPoint, centerY - 5);