<ListView Margin="10" BorderThickness="0" Background="Transparent">
  <ListView.ItemsPanel>
    <ItemsPanelTemplate><WrapPanel/></ItemsPanelTemplate>
  </ListView.ItemsPanel>

  <ListView.ItemTemplate>
    <DataTemplate>
      <Border Width="140" Height="130" Margin="6" Background="White" BorderBrush="#330" BorderThickness="1">
        <StackPanel>
          <TextBlock Text="{Binding Name}" FontWeight="Bold" FontSize="12" Margin="0,6,0,4" HorizontalAlignment="Center"/>
          <Image Source="{Binding Icon}" Width="120" Height="70" Stretch="UniformToFill"/>
        </StackPanel>
      </Border>
    </DataTemplate>
  </ListView.ItemTemplate>

  <!-- 直接在 XAML 塞兩個 item 測試 -->
  <ListViewItem>
    <ListViewItem.DataContext>
      <sys:Object xmlns:sys="clr-namespace:System;assembly=mscorlib">
        <sys:DynamicObject/>
      </sys:Object>
    </ListViewItem.DataContext>
  </ListViewItem>
</ListView>