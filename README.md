xmlns:diag="clr-namespace:System.Diagnostics;assembly=WindowsBase"

<ListBox ItemsSource="{Binding Patterns}">
  <ListBox.ItemTemplate>
    <DataTemplate>
      <StackPanel Orientation="Horizontal" Margin="6">
        <!-- 直接顯示 Icon 字串，確認路徑內容 -->
        <TextBlock Text="{Binding Icon}" Width="420" Margin="0,0,8,0"/>

        <!-- 圖片：開啟綁定追蹤 + 失敗事件 -->
        <Image Width="64" Height="64" Stretch="UniformToFill" Margin="0,0,8,0"
               Source="{Binding Icon, diag:PresentationTraceSources.TraceLevel=High}"
               ImageFailed="PatternImage_ImageFailed"/>
        <TextBlock Text="{Binding Name}" VerticalAlignment="Center"/>
      </StackPanel>
    </DataTemplate>
  </ListBox.ItemTemplate>
</ListBox>



private void PatternImage_ImageFailed(object sender, ExceptionRoutedEventArgs e)
{
    var fe = sender as FrameworkElement;
    var icon = fe?.DataContext?
        .GetType()
        .GetProperty("Icon")
        ?.GetValue(fe.DataContext) as string;

    System.Diagnostics.Debug.WriteLine("❌ ImageFailed: " + (icon ?? "(null)") + " | " + e.ErrorException?.Message);
    System.Windows.MessageBox.Show(
        "Image load failed:\n" + (icon ?? "(null)") + 
        "\n\n" + (e.ErrorException?.Message ?? "(no message)"));
}


System.Diagnostics.Debug.WriteLine("🟦 iconUri = " + iconUri);
