<ListBox x:Name="lb"
         ItemsSource="{Binding}" 
         SelectionMode="Single" 
         ScrollViewer.HorizontalScrollBarVisibility="Disabled">
  <ListBox.ItemsPanel>
    <ItemsPanelTemplate>
      <WrapPanel Orientation="Horizontal"/>
    </ItemsPanelTemplate>
  </ListBox.ItemsPanel>

  <ListBox.ItemTemplate>
    <DataTemplate>
      <Border Width="140" Height="132" Margin="6" Padding="8" BorderThickness="1">
        <Border.Style>
          <Style TargetType="Border">
            <Setter Property="BorderBrush" Value="#DDD"/>
            <Setter Property="Background" Value="#FFF"/>
            <Style.Triggers>
              <DataTrigger Binding="{Binding IsSelected, RelativeSource={RelativeSource AncestorType=ListBoxItem}}" Value="True">
                <Setter Property="BorderBrush" Value="#4A90E2"/>
                <Setter Property="Background" Value="#EEF5FF"/>
              </DataTrigger>
              <Trigger Property="IsMouseOver" Value="True">
                <Setter Property="BorderBrush" Value="#999"/>
              </Trigger>
            </Style.Triggers>
          </Style>
        </Border.Style>

        <StackPanel>
          <!-- 解碼寬可大幅降記憶體 -->
          <Image Source="{Binding ImagePath}" Height="90" Stretch="UniformToFill"
                 RenderOptions.BitmapScalingMode="Fant"
                 SnapsToDevicePixels="True"/>
          <TextBlock Text="{Binding Name}" Margin="0,6,0,0"
                     HorizontalAlignment="Center"
                     TextTrimming="CharacterEllipsis"/>
        </StackPanel>
      </Border>
    </DataTemplate>
  </ListBox.ItemTemplate>
</ListBox>