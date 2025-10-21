<GroupBox Header="{Binding AutoRunVM.Title}"
          Visibility="{Binding IsAutoRunConfigVisible, Converter={StaticResource BoolToVisibilityConverter}}">
  <!-- ← 外層 DataContext 是 BistModeViewModel -->

  <StackPanel x:Name="AutoRunPanel" Margin="12" DataContext="{Binding AutoRunVM}">
    <!-- (1) 總張數 -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Pattern Total:" VerticalAlignment="Center" Margin="0,0,8,0"/>
      <ComboBox Width="96"
                ItemsSource="{Binding TotalOptions}"
                SelectedItem="{Binding Total, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- (2) 動態的圖片順序 -->
    <ItemsControl ItemsSource="{Binding Orders}">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <StackPanel Orientation="Horizontal" Margin="0,2">
            <TextBlock Width="110" VerticalAlignment="Center"
                       Text="{Binding DisplayNo, StringFormat=圖片順序 {0}}"/>

            <!-- ✅ 改用 ElementName=AutoRunPanel 取 AutoRunVM 的清單 -->
            <ComboBox Width="160"
                      ItemsSource="{Binding DataContext.PatternIndexOptions,
                                            ElementName=AutoRunPanel}"
                      SelectedItem="{Binding SelectedIndex, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- (3) 套用 -->
    <Button Content="Set Auto-Run"
            Width="120" Height="30" Margin="0,12,0,0"
            Command="{Binding ApplyCommand}"/>
  </StackPanel>
</GroupBox>