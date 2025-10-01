<Window x:Class="BistApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="BIST Pattern List" Height="620" Width="880"
        Background="#FFF2F2F2">
  <Grid Margin="12">
    <ListView ItemsSource="{Binding Patterns}"
              BorderThickness="0"
              Background="Transparent"
              ScrollViewer.VerticalScrollBarVisibility="Auto"
              SelectionMode="Single">

      <!-- 讓清單變成「圖塊牆」排列 -->
      <ListView.ItemsPanel>
        <ItemsPanelTemplate>
          <WrapPanel IsItemsHost="True"/>
        </ItemsPanelTemplate>
      </ListView.ItemsPanel>

      <!-- 每一個 Pattern 的外觀 -->
      <ListView.ItemTemplate>
        <DataTemplate>
          <Border Width="140" Height="130" Margin="8" CornerRadius="6"
                  Background="White" BorderBrush="#33000000" BorderThickness="1">
            <StackPanel>
              <TextBlock Text="{Binding DisplayName}"
                         FontWeight="Bold" FontSize="12"
                         Margin="0,6,0,4" HorizontalAlignment="Center"/>

              <Border Margin="8,0,8,0" BorderBrush="#22000000" BorderThickness="1">
                <Image Source="{Binding Icon}" Width="120" Height="70"
                       Stretch="UniformToFill"
                       RenderOptions.BitmapScalingMode="HighQuality"
                       SnapsToDevicePixels="True"/>
              </Border>

              <TextBlock Text="{Binding Name}" FontSize="11"
                         Margin="0,4,0,0" HorizontalAlignment="Center"
                         TextTrimming="CharacterEllipsis"/>
            </StackPanel>
          </Border>
        </DataTemplate>
      </ListView.ItemTemplate>

      <!-- 選取時的外框顏色（好辨識） -->
      <ListView.ItemContainerStyle>
        <Style TargetType="ListViewItem">
          <Setter Property="Padding" Value="0"/>
          <Setter Property="HorizontalContentAlignment" Value="Stretch"/>
          <Setter Property="VerticalContentAlignment" Value="Stretch"/>
          <Setter Property="BorderThickness" Value="0"/>
          <Style.Triggers>
            <Trigger Property="IsSelected" Value="True">
              <Setter Property="Effect">
                <Setter.Value>
                  <DropShadowEffect ShadowDepth="0" BlurRadius="10" Opacity="0.6"/>
                </Setter.Value>
              </Setter>
              <Setter Property="Foreground" Value="Black"/>
            </Trigger>
          </Style.Triggers>
        </Style>
      </ListView.ItemContainerStyle>

    </ListView>
  </Grid>
</Window>