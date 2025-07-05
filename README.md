public partial class DriverPage : Page, IKnobHandler
{
    private bool isPoiVisible = false;

    public DriverPage()
    {
        InitializeComponent();
        PoiCardContainer.Visibility = Visibility.Collapsed;
    }

    public void OnDriverKnobRotated(KnobEvent e)
    {
        // 先留空或做其他 rotate 功能
    }

    public void OnPassengerKnobRotated(KnobEvent e)
    {
        // 同上
    }

    public void OnDriverKnobPressed(KnobEvent e)
    {
        // 同上
    }

    public void OnPassengerKnobPressed(KnobEvent e)
    {
        // 🎯 這裡加入呼叫 POI 方法
        TogglePOICard(showOnRight: true);
    }

    private void TogglePOICard(bool showOnRight)
    {
        isPoiVisible = !isPoiVisible;

        if (isPoiVisible)
        {
            PoiCardContainer.Visibility = Visibility.Visible;

            if (showOnRight)
            {
                PoiCardContainer.HorizontalAlignment = HorizontalAlignment.Right;
                PoiCardContainer.Margin = new Thickness(0, 300, 50, 0);
            }
            else
            {
                PoiCardContainer.HorizontalAlignment = HorizontalAlignment.Left;
                PoiCardContainer.Margin = new Thickness(50, 300, 0, 0);
            }
        }
        else
        {
            PoiCardContainer.Visibility = Visibility.Collapsed;
        }
    }

    private void TogglePOI_Click(object sender, RoutedEventArgs e)
    {
        TogglePOICard(showOnRight: true);
    }
}