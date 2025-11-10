<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Select Image" Width="720" Height="560"
        WindowStartupLocation="CenterOwner">
  <DockPanel Margin="12">
    <ScrollViewer VerticalScrollBarVisibility="Auto">
      <ItemsControl x:Name="ic">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <WrapPanel Orientation="Horizontal"/>
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Border Margin="6" Padding="8" BorderBrush="#DDD" BorderThickness="1"
                    Width="160" Height="130" Cursor="Hand"
                    MouseLeftButtonUp="OnPick">
              <StackPanel>
                <Image Source="{Binding ImagePath}" Height="90" Stretch="UniformToFill"/>
                <TextBlock Text="{Binding Name}" Margin="0,6,0,0"
                           TextTrimming="CharacterEllipsis"
                           HorizontalAlignment="Center"/>
              </StackPanel>
            </Border>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </ScrollViewer>

    <StackPanel DockPanel.Dock="Bottom" Orientation="Horizontal"
                HorizontalAlignment="Right" Margin="0,8,0,0">
      <Button Content="取消" Width="90" IsCancel="True"/>
    </StackPanel>
  </DockPanel>
</Window>