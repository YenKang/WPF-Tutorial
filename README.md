
```xaml
<Grid>
    <Image x:Name="DriverMapImage"
           Width="1600"
           Height="900"
           Source="/NovaCID;component/Assets/DriverMaps/Normal.png"
           Stretch="Uniform"/>
</Grid>
```


```csharp
private void LoadMap()
{
    if (_currentMapIndex < 0) _currentMapIndex = 0;
    if (_currentMapIndex >= _mapPaths.Length) _currentMapIndex = _mapPaths.Length - 1;

    DriverMapImage.Source = new BitmapImage(new Uri(_mapPaths[_currentMapIndex], UriKind.Absolute));
}
```