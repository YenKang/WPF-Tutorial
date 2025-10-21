<!-- 🧭 Auto-Run 設定區：根據是否選中 autoRunControl 圖片顯示 -->
<GroupBox
    Header="{Binding AutoRunVM.Title}"
    Visibility="{Binding IsAutoRunConfigVisible, Converter={StaticResource BoolToVisibilityConverter}}"
    Margin="0,12,0,0">

  <!-- 設定內層 DataContext，讓所有綁定直接使用 AutoRunVM -->
  <StackPanel Margin="12" DataContext="{Binding AutoRunVM}">

    <!-- (1) 總張數設定：控制 Orders 的數量 -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Pattern Total:" VerticalAlignment="Center" Margin="0,0,8,0"/>
      <ComboBox Width="96"
                ItemsSource="{Binding TotalOptions}"
                SelectedItem="{Binding Total, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
    </StackPanel>

    <!-- (2) 動態生成 N 個「圖片順序」下拉列 -->
    <ItemsControl ItemsSource="{Binding Orders}">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <StackPanel Orientation="Horizontal" Margin="0,2">
            <!-- 左：顯示圖片順序（1,2,3...） -->
            <TextBlock Width="110" VerticalAlignment="Center"
                       Text="{Binding DisplayNo, StringFormat=圖片順序 {0}}"/>

            <!-- 右：可選 pattern index 清單 -->
            <ComboBox Width="160"
                      ItemsSource="{Binding DataContext.PatternIndexOptions,
                                            RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      SelectedItem="{Binding SelectedIndex, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- (3) 套用按鈕：將設定寫入暫存器 -->
    <Button Content="Set Auto-Run"
            Width="120" Height="30" Margin="0,12,0,0"
            Command="{Binding ApplyCommand}"/>
  </StackPanel>
</GroupBox>