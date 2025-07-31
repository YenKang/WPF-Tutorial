

// /ViewModel/DrivePageViewModel.cs
using System.Collections.Generic;

public partial class DrivePageViewModel : IFingerHandler
{
    public void OnFingerTouched(List<FingerStatus> fingers)
    {
        foreach (var finger in fingers)
        {
            if (!finger.IsDriver) continue;

            if (IsInsideZoomInButton(finger.X, finger.Y))
            {
                ZoomLevel++;
            }
            else if (IsInsideZoomOutButton(finger.X, finger.Y))
            {
                ZoomLevel--;
            }
        }
    }

    private bool IsInsideZoomInButton(int x, int y)
    {
        // TODO: 根據 Zoom In Button 的邊界位置實作
        return (x >= 1200 && x <= 1300 && y >= 1000 && y <= 1100);
    }

    private bool IsInsideZoomOutButton(int x, int y)
    {
        return (x >= 1400 && x <= 1500 && y >= 1000 && y <= 1100);
    }
}

