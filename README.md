<!-- 3-6) Auto Run Control -->
<GroupBox
    Header="{Binding AutoRunVM.Title}"
    Visibility="{Binding IsAutoRunConfigVisible,
                         Converter={utilityConv:BoolToVisibilityConverter}}"
    Margin="0,12,0,0">

    <!-- 內層 DataContext = AutoRunVM -->
    <StackPanel x:Name="AutoRunPanel"
                Margin="12"
                DataContext="{Binding AutoRunVM}">

        <!-- (1) Pattern Total：UpDown 1~22 -->
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

        <!-- (2) Pattern Orders：固定 N 列（JSON 有幾筆 patternOrder 就幾個） -->
        <ItemsControl ItemsSource="{Binding Orders}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Margin="0,2,0,2">

                        <!-- 顯示 Pattern Order 編號 -->
                        <TextBlock Width="120"
                                   VerticalAlignment="Center"
                                   Text="{Binding DisplayNo, StringFormat=Pattern Order: {0}}" />

                        <!-- 顯示圖片名稱，下暫存器用 Index（SelectedPatternNumber） -->
                        <ComboBox Width="160"
                                  ItemsSource="{Binding DataContext.PatternOptions,
                                                        ElementName=AutoRunPanel}"
                                  DisplayMemberPath="Display"
                                  SelectedValuePath="Index"
                                  SelectedValue="{Binding SelectedPatternNumber,
                                                          Mode=TwoWay,
                                                          UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>

        <!-- (3) 寫入 Auto Run -->
        <Button Content="Set Auto Run"
                Width="140"
                Height="32"
                HorizontalAlignment="Left"
                Margin="0,12,0,0"
                Command="{Binding ApplyCommand}" />

    </StackPanel>
</GroupBox>
