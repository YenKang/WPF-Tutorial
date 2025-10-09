<ListBox ItemsSource="{Binding Patterns}"
         SelectedItem="{Binding SelectedPattern, Mode=TwoWay}"
         SelectionMode="Single"
         BorderThickness="0"
         ScrollViewer.HorizontalScrollBarVisibility="Disabled">

    <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel/>
        </ItemsPanelTemplate>
    </ListBox.ItemsPanel>

    <ListBox.ItemTemplate>
        <DataTemplate>
            <Border Width="70" Height="60"
                    BorderThickness="1"
                    BorderBrush="Gray"
                    CornerRadius="3"
                    Margin="3"
                    Padding="2">
                <StackPanel VerticalAlignment="Center">
                    <!-- 上方：P 編號 + 勾選框 -->
                    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center">
                        <TextBlock Text="{Binding Index, StringFormat=P{0}}"
                                   VerticalAlignment="Center"
                                   Margin="0,0,6,0"
                                   FontSize="12"/>
                        <Border Width="12" Height="12" BorderThickness="1" BorderBrush="Gray">
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Style.Triggers>
                                        <DataTrigger Binding="{Binding IsSelected,
                                            RelativeSource={RelativeSource AncestorType=ListBoxItem}}"
                                                     Value="True">
                                            <Setter Property="Background" Value="#E0E0E0"/>
                                        </DataTrigger>
                                    </Style.Triggers>
                                </Style>
                            </Border.Style>
                        </Border>
                    </StackPanel>

                    <!-- 下方：Button -->
                    <Button Content="SET"
                            Margin="0,6,0,0"
                            FontSize="10"
                            Padding="2"
                            HorizontalAlignment="Center"
                            Command="{Binding DataContext.ApplyPatternCommand, RelativeSource={RelativeSource AncestorType=ListBox}}"
                            CommandParameter="{Binding}" />
                </StackPanel>
            </Border>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>