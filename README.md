<GroupBox Header="Pattern En" Margin="16">
    <ListBox ItemsSource="{Binding Patterns}"
             SelectedItem="{Binding SelectedPattern, Mode=TwoWay}"
             SelectionMode="Single"
             BorderThickness="0"
             HorizontalContentAlignment="Stretch">
        <!-- 讓項目自動換行成多欄 -->
        <ListBox.ItemsPanel>
            <ItemsPanelTemplate>
                <WrapPanel />
            </ItemsPanelTemplate>
        </ListBox.ItemsPanel>

        <!-- 每個項目的外觀： P0 ☐、P1 ☐ ... -->
        <ListBox.ItemTemplate>
            <DataTemplate>
                <Border Padding="6" Margin="6" CornerRadius="4" Background="#1AFFFFFF">
                    <StackPanel Orientation="Horizontal" VerticalAlignment="Center">
                        <TextBlock Text="{Binding Name}" Width="36" Margin="0,0,6,0"/>
                        <!-- 勾選狀態 = 該 ListBoxItem 是否被選取（唯一性由 ListBox 管） -->
                        <CheckBox IsChecked="{Binding IsSelected,
                                   RelativeSource={RelativeSource AncestorType=ListBoxItem},
                                   Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                    </StackPanel>
                </Border>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
</GroupBox>