public void OnFingerTouch(List<FingerStatus> fingersList)
{
    foreach (var finger in fingersList)
    {
        if (finger.Status != "Down") continue;           // 條件 1
        if (!finger.Driver) continue;                    // 條件 2

        var point = new Point(finger.X, finger.Y);

        if (IsPointInArea(point, ZoomInBtnX, ZoomInBtnY, ZoomInWidth, ZoomInHeight)) // 條件 3
        {
            Debug.WriteLine("✅ 主駕駛按下 ZoomIn 區域");
            SetZoomLevel(ZoomLevel + 1);
        }
    }
}

public bool IsPointInArea(Point p, double x, double y, double width, double height)
{
    return new Rect(x, y, width, height).Contains(p);
}