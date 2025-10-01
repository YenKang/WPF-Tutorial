<ListView ItemsSource="{Binding Patterns}"
          Margin="10" BorderThickness="0" Background="Transparent">
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
</ListView>