<!-- ===================== Auto Run ===================== -->
<GroupBox Header="Auto Run"
         Margin="0,12,0,0"
         Visibility="{Binding IsAutoRunConfigVisible,
                              Converter={StaticResource BoolToVisibilityConverter}}">
  <StackPanel Margin="12" Orientation="Vertical" >

    <!-- Total (1~22) -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Total images:" VerticalAlignment="Center" Margin="0,0,8,0"/>
      <ComboBox Width="80"
                ItemsSource="{Binding AutoRunVM.TotalOptions}"
                SelectedItem="{Binding AutoRunVM.Total, Mode=TwoWay}"/>
    </StackPanel>

    <!-- Orders（動態產生 ORD0..ORD(N-1)；每列一個 ComboBox） -->
    <ItemsControl ItemsSource="{Binding AutoRunVM.Orders}"
                  AlternationCount="100">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <Grid Margin="0,2">
            <Grid.ColumnDefinitions>
              <ColumnDefinition Width="120"/>
              <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>

            <!-- 左邊標籤：ORD# -->
            <TextBlock Grid.Column="0"
                       VerticalAlignment="Center"
                       Text="{Binding RelativeSource={RelativeSource AncestorType=ItemsControl},
                                      Path=(ItemsControl.AlternationIndex),
                                      StringFormat=ORD{0}}"/>

            <!-- 右邊：選圖（SelectedItem 綁定到目前這個 Orders 的元素值本身） -->
            <ComboBox Grid.Column="1" Width="160"
                      ItemsSource="{Binding DataContext.AutoRunVM.PatternIndexOptions,
                                            RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      SelectedItem="{Binding ., Mode=TwoWay}"
                      ToolTip="Pattern index (0~63), 依實際提供的清單為準"/>
          </Grid>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <Separator Margin="0,10,0,10"/>

    <!-- FCNT1 / FCNT2 -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
      <TextBlock Text="FCNT1:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <TextBox Width="80"
               Text="{Binding AutoRunVM.FCNT1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
               ToolTip="建議：0x3C=60 frames（1 sec）"/>
      <TextBlock Text="   FCNT2:" VerticalAlignment="Center" Margin="12,0,6,0"/>
      <TextBox Width="80"
               Text="{Binding AutoRunVM.FCNT2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
               ToolTip="建議：0x1E=30 frames（0.5 sec）"/>
    </StackPanel>

    <!-- 套用 -->
    <Button Content="Set"
            Width="90" Height="30"
            HorizontalAlignment="Left"
            Command="{Binding AutoRunVM.ApplyCommand}"/>
  </StackPanel>
</GroupBox>