<DataTemplate>
    <StackPanel Margin="4">
        <!-- 這裡固定格子大小 -->
        <Border Width="80" Height="80"
                Background="Black"      <!-- 想透明就改 Transparent -->
                ClipToBounds="True">
            <Image Source="{Binding Thumbnail}"
                   Stretch="Uniform"    <!-- 等比例縮放，不變形 -->
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   RenderOptions.BitmapScalingMode="HighQuality"
                   SnapsToDevicePixels="True" />
        </Border>

        <!-- 底下原本就有的文字，可以保持不動 -->
        <TextBlock Text="{Binding DisplayName}"
                   HorizontalAlignment="Center"
                   Margin="0,4,0,0" />
    </StackPanel>
</DataTemplate>
