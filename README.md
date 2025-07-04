<Image x:Name="POICardImage"
       Source="/NovaCID;component/Assets/DriverPage/POICard.png"
       Width="450"
       Height="450"
       Visibility="Collapsed"
       HorizontalAlignment="Right"
       VerticalAlignment="Top"
       Margin="20"/>

```csharp
private bool _isPoiCardVisible = false;

public void TogglePoiCard()
{
    _isPoiCardVisible = !_isPoiCardVisible;
    PoiCardImage.Visibility = _isPoiCardVisible ? Visibility.Visible : Visibility.Collapsed;
}

private void TestTogglePoi_Click(object sender, RoutedEventArgs e)
{
    TogglePoiCard();
}
```

```xml
<Button Content="Toggle POI" Click="TestTogglePoi_Click" Width="200" Height="60"/>
```