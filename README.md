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

  <!-- 正確：直接放資料物件 -->
  <ListView.Items>
    <local:PatternItem Name="Black" Icon="Assets/ㄇ<ListView Margin="10" BorderThickness="0" Background="Transparent">
  <ListView.ItemsPanel>
    <ItemsPanelTemplate><WrapPanel/></ItemsPanelTemplate>
  </ListView.ItemsPanel>

  <ListView.ItemTemplate>
    <DataTemplate>
      <StackPanel Width="140" Margin="6">
        <Image Source="{Binding Icon}" Width="120" Height="70" Stretch="UniformToFill"/>
        <TextBlock Text="{Binding Name}" HorizontalAlignment="Center" Margin="0,6,0,0"/>
      </StackPanel>
    </DataTemplate>
  </ListView.ItemTemplate>

  <ListView.Items>
    <local:PatternItem Name="Black" Icon="pack://siteoforigin:,,,/Assets/Patterns/NT51635/PatIdx_0_Black.png"/>
    <local:PatternItem Name="White" Icon="pack://siteoforigin:,,,/Assets/Patterns/NT51635/PatIdx_1_White.png"/>
  </ListView.Items>
</ListView>Patterns/NT51635/PatIdx_0_Black.png"/>
    <local:PatternItem Name="White" Icon="Assets/Patterns/NT51635/PatIdx_1_White.png"/>
  </ListView.Items>
</ListView>