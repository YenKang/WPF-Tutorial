<!-- 3-7) Auto Run Control -->
<GroupBox Header="{Binding AutoRunVM.Title}"
          Visibility="{Binding IsAutoRunConfigVisible, Converter={utilityConv:BoolToVisibilityConverter}}"
          Margin="0,12,0,0">
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
                       Text="{Binding DisplayNo, StringFormat=Pattern Order {0}}"/>
            <!-- 從 GroupBox 找到 AutoRunVM.PatternOptions 作為清單 -->
            <ComboBox Width="220"
                      ItemsSource="{Binding DataContext.AutoRunVM.PatternOptions,
                                            RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      DisplayMemberPath="Display"
                      SelectedValuePath="Index"
                      SelectedValue="{Binding SelectedIndex, Mode=TwoWay}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <Button Content="Set Auto-Run"
            Margin="0,12,0,0"
            Command="{Binding ApplyCommand}"/>
  </StackPanel>
</GroupBox>