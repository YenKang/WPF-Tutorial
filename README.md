public partial class NovaCIDView : UserControl
{
    public NovaCIDView()
    {
        InitializeComponent();
        PositionLeftKnob(); // ✅ 呼叫你的方法

        // 模擬螢幕視窗大小（可選）
        var win = Window.GetWindow(this);
        if (win != null)
        {
            win.Width = 1472; // 模擬 15.6"
            win.Height = 829;
            win.WindowStartupLocation = WindowStartupLocation.CenterScreen;
        }
    }

    private void PositionLeftKnob()
    {
        double centerX = 445;
        double centerY = 1113; // ✅ 根據 15 cm 測量

        double outerRadius = 186;
        double innerRadius = 112;

        Canvas.SetLeft(LeftKnobOuter, centerX - outerRadius);
        Canvas.SetTop(LeftKnobOuter, centerY - outerRadius);

        Canvas.SetLeft(LeftKnobInner, centerX - innerRadius);
        Canvas.SetTop(LeftKnobInner, centerY - innerRadius);
    }
}