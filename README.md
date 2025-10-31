<!-- (2) 動態的圖片順序 -->
<ItemsControl ItemsSource="{Binding Orders}">
  <ItemsControl.ItemTemplate>
    <DataTemplate>
      <StackPanel Orientation="Horizontal" Margin="0,2">
        <TextBlock Width="110" VerticalAlignment="Center"
                   Text="{Binding DisplayNo, StringFormat=Pattern Order {0}}" />

        <!-- 用 ElementName=AutoRunPanel 靠住上層 GroupBox 的 DataContext -->
        <ComboBox Width="160"
                  ItemsSource="{Binding ElementName=AutoRunPanel,
                                        Path=DataContext.AutoRunVM.PatternOptions}"
                  DisplayMemberPath="Display"
                  SelectedValuePath="Index"
                  SelectedValue="{Binding SelectedIndex,
                                          Mode=TwoWay,
                                          UpdateSourceTrigger=PropertyChanged,
                                          ValidatesOnDataErrors=False,
                                          ValidatesOnExceptions=False,
                                          NotifyOnValidationError=False}" />
      </StackPanel>
    </DataTemplate>
  </ItemsControl.ItemTemplate>
</ItemsControl>