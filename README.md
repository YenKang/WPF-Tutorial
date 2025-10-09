<GroupBox Header="Pattern En" Margin="12">
    <ListBox ItemsSource="{Binding Patterns}"
             SelectedItem="{Binding SelectedPattern, Mode=TwoWay}"
             SelectionMode="Single"
             BorderThickness="0"
             ScrollViewer.HorizontalScrollBarVisibility="Disabled">
        <!-- 讓項目自動排列換行 -->
        <ListBox.ItemsPanel>
            <ItemsPanelTemplate>
                <WrapPanel Orientation="Horizontal" />
            </ItemsPanelTemplate>
        </ListBox.ItemsPanel>

        <ListBox.ItemTemplate>
            <DataTemplate>
                <Border Width="65" Height="30" Margin="2"
                        CornerRadius="4"
                        BorderBrush="#555" BorderThickness="1"
                        Background="{Binding IsSelected, 
                                     RelativeSource={RelativeSource AncestorType=ListBoxItem},
                                     Converter={StaticResource BoolToBrushConverter}}">
                    <TextBlock Text="{Binding Name}"
                               VerticalAlignment="Center"
                               HorizontalAlignment="Center"
                               Foreground="White"
                               FontSize="14"/>
                </Border>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
</GroupBox>