if (IsInButtonArea(p, ZoomInBtnX, ZoomInBtnY, ZoomInWidth, ZoomInHeight))
{
    ExecuteZoomIn();
}
else if (IsInButtonArea(p, ZoomOutBtnX, ZoomOutBtnY, ZoomOutWidth, ZoomOutHeight))
{
    ExecuteZoomOut();
}

private bool IsInButtonArea(Point p, double x, double y, double width, double height)
{
    return new Rect(x, y, width, height).Contains(p);
}