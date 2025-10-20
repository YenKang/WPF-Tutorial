<!-- 可重用：底線樣式（只畫下邊線） -->
<Style x:Key="BottomLineBase" TargetType="Border">
  <Setter Property="BorderThickness" Value="0,0,0,2"/>
  <Setter Property="Margin" Value="2"/>
</Style>

<Style x:Key="BottomLineRed" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Red"/>
</Style>
<Style x:Key="BottomLineGreen" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Green"/>
</Style>
<Style x:Key="BottomLineBlue" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Blue"/>
</Style>

==========


<GroupBox Header="Flicker Plane RGB Level"
          Visibility="{Binding IsFlickerRGBVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,12,0,0">
  <Grid Margin="8">
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto"/>   <!-- 標題列 -->
      <RowDefinition Height="*"/>      <!-- ItemsControl -->
    </Grid.RowDefinitions>

    <!-- 標題列 -->
    <Grid Grid.Row="0">
      <Grid.ColumnDefinitions>
        <ColumnDefinition Width="40"/>
        <ColumnDefinition Width="80"/>
        <ColumnDefinition Width="80"/>
        <ColumnDefinition Width="80"/>
        <ColumnDefinition Width="*"/>
      </Grid.ColumnDefinitions>
      <TextBlock Grid.Column="0"/>
      <TextBlock Grid.Column="1" Text="R" FontWeight="Bold" HorizontalAlignment="Center"/>
      <TextBlock Grid.Column="2" Text="G" FontWeight="Bold" HorizontalAlignment="Center"/>
      <TextBlock Grid.Column="3" Text="B" FontWeight="Bold" HorizontalAlignment="Center"/>
    </Grid>

    <!-- P1~P4：每列一個 PlaneVM（R/G/B 三個下底線色） -->
    <ItemsControl Grid.Row="1" ItemsSource="{Binding FlickerRGBVM.Planes}">
      <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
          <StackPanel/>
        </ItemsPanelTemplate>
      </ItemsControl.ItemsPanel>

      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Grid Margin="0,2,0,2">
            <Grid.ColumnDefinitions>
              <ColumnDefinition Width="40"/>
              <ColumnDefinition Width="80"/>
              <ColumnDefinition Width="80"/>
              <ColumnDefinition Width="80"/>
            </Grid.ColumnDefinitions>

            <!-- 左側：P1/P2/P3/P4 標籤（用 Index 屬性顯示） -->
            <TextBlock Grid.Column="0"
                       Text="{Binding Index, StringFormat=P{0}}"
                       VerticalAlignment="Center"/>

            <!-- R（紅底線） -->
            <Border Grid.Column="1" Style="{StaticResource BottomLineRed}">
              <ComboBox ItemsSource="{Binding LevelOptions}"
                        SelectedItem="{Binding R, Mode=TwoWay}"
                        ItemStringFormat="{}{0:X}"
                        Width="70"/>
            </Border>

            <!-- G（綠底線） -->
            <Border Grid.Column="2" Style="{StaticResource BottomLineGreen}">
              <ComboBox ItemsSource="{Binding LevelOptions}"
                        SelectedItem="{Binding G, Mode=TwoWay}"
                        ItemStringFormat="{}{0:X}"
                        Width="70"/>
            </Border>

            <!-- B（藍底線） -->
            <Border Grid.Column="3" Style="{StaticResource BottomLineBlue}">
              <ComboBox ItemsSource="{Binding LevelOptions}"
                        SelectedItem="{Binding B, Mode=TwoWay}"
                        ItemStringFormat="{}{0:X}"
                        Width="70"/>
            </Border>
          </Grid>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- 右側 Set 按鈕（跨整個 ItemsControl 區塊置中） -->
    <Button Content="Set"
            Command="{Binding FlickerRGBVM.ApplyCommand}"
            Width="80" Height="30"
            HorizontalAlignment="Right" VerticalAlignment="Center"
            Grid.RowSpan="2" Margin="8,0,0,0"/>
  </Grid>
</GroupBox>