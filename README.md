<!-- 🔹Auto-Run 設定 GroupBox -->
<GroupBox Header="Auto-Run Config"
          Visibility="{Binding IsAutoRunConfigVisible,
                               Converter={StaticResource BoolToVisibilityConverter}}"
          Margin="0,12,0,0"
          DataContext="{Binding AutoRunVM}">
  <StackPanel Margin="8">

    <!-- 🟢 Auto-Run Pattern Total -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Pattern Total:" VerticalAlignment="Center" Width="120"/>
      <ComboBox Width="100"
                ItemsSource="{Binding TotalOptions}"
                SelectedItem="{Binding Total, Mode=TwoWay}"/>
    </StackPanel>

    <!-- 🟢 Flicker Counter (FCNT1 / FCNT2) -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="FCNT1:" VerticalAlignment="Center" Width="60"/>
      <TextBox Width="80" Text="{Binding Fcnt1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
      <TextBlock Text="   FCNT2:" VerticalAlignment="Center"/>
      <TextBox Width="80" Text="{Binding Fcnt2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- 🟢 Auto-Run Orders (依 Pattern 數量產生) -->
    <ItemsControl ItemsSource="{Binding Orders}">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <StackPanel Orientation="Horizontal" Margin="0,2">
            <TextBlock Text="{Binding Index, StringFormat='ORD{0:D2}:'}" Width="70" VerticalAlignment="Center"/>
            <ComboBox Width="160"
                      ItemsSource="{Binding DataContext.PatternIndexOptions,
                                            RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      SelectedItem="{Binding SelectedIndex, Mode=TwoWay}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- 🟢 Apply 按鈕 -->
    <Button Content="Set Auto-Run"
            Command="{Binding ApplyCommand}"
            Width="140" Height="30"
            HorizontalAlignment="Left"
            Margin="0,10,0,0"/>
  </StackPanel>
</GroupBox>