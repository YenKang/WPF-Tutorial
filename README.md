public void SetZoomButtons(Rect zoomInArea, Rect zoomOutArea)
{
    _zoomInArea = zoomInArea;
    _zoomOutArea = zoomOutArea;

    Console.WriteLine($"[Debug] ZoomInArea: X={_zoomInArea.X}, Y={_zoomInArea.Y}, W={_zoomInArea.Width}, H={_zoomInArea.Height}");
}

