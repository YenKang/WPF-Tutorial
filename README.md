

```
<DataTemplate>
    <Border Width="80" Height="80"
            Background="Black"   <!-- 可改成 Transparent -->
            ClipToBounds="True"
            Margin="4">
        <Image Source="{Binding Thumbnail}"
               Stretch="Uniform"
               HorizontalAlignment="Center"
               VerticalAlignment="Center"
               RenderOptions.BitmapScalingMode="HighQuality"
               SnapsToDevicePixels="True" />
    </Border>
</DataTemplate>
```
