<!-- 3-6) Auto Run Control -->
<GroupBox
    Header="{Binding AutoRunVM.Title}"
    Visibility="{Binding IsAutoRunConfigVisible,
                         Converter={utilityConv:BoolToVisibilityConverter}}"
    Margin="0,12,0,0">

    <!-- 這裡沿用舊格式：內層 StackPanel 的 DataContext = AutoRunVM -->
    <StackPanel x:Name="AutoRunPanel"
                Margin="12"
                DataContext="{Binding AutoRunVM}">

        <!-- (1) Pattern Total：改成 UpDown，範圍 1~22 -->
        <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
            <TextBlock Text="Pattern Total"
                       VerticalAlignment="Center"
                       Margin="0,0,8,0" />

            <xctk:IntegerUpDown Width="96"
                                Minimum="1"
                                Maximum="22"
                                Value="{Binding Total,
                                                Mode=TwoWay,
                                                UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- (2) 固定的圖片順序：JSON 有幾筆 patternOrder，就有幾個 combobox（通常 22） -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,2,0,2">

                        <!-- 顯示 Pattern Order 編號 -->
                        <TextBlock Width="110"
                                   VerticalAlignment="Center"
                                   Text="{Binding DisplayNo, StringFormat=Pattern Order: {0}}" />

                        <!-- 每列選圖片編號（1~22），直接當暫存器值 -->
                        <ComboBox Width="96"
                                  ItemsSource="{Binding DataContext.PatternNumberOptions,
                                                        ElementName=AutoRunPanel}"
                                  SelectedItem="{Binding SelectedPatternNumber,
                                                         Mode=TwoWay,
                                                         UpdateSourceTrigger=PropertyChanged}" />
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <!-- 寫入 Auto Run -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Margin="0,12,0,0"
                Command="{Binding ApplyCommand}" />

    </StackPanel>
</GroupBox>
