<GroupBox Header="Pattern En" Margin="12">
  <DockPanel>
    <!-- 上方控制列：調整每列幾個 -->
    <StackPanel Orientation="Horizontal" DockPanel.Dock="Top" Margin="0,0,0,8">
      <TextBlock Text="Columns:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <TextBox Width="40" Text="{Binding Columns, UpdateSourceTrigger=PropertyChanged}"/>
      <!-- 如果你想改成固定列數，下面把 Columns 換 Rows 就好 -->
    </StackPanel>

    <ListBox ItemsSource="{Binding Patterns}"
             SelectedItem="{Binding SelectedPattern}"
             ScrollViewer.HorizontalScrollBarVisibility="Disabled"
             BorderThickness="0"
             HorizontalContentAlignment="Stretch">
      <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
          <!-- 指定幾欄，就會自動換行 -->
          <UniformGrid Columns="{Binding Columns}"/>
          <!-- 若想指定幾列，改成 Rows="{Binding Rows}" -->
        </ItemsPanelTemplate>
      </ListBox.ItemsPanel>

      <ListBox.ItemTemplate>
        <DataTemplate>
          <Border Padding="6" Margin="4" Background="#1AFFFFFF" CornerRadius="4">
            <StackPanel Orientation="Horizontal">
              <TextBlock Text="{Binding Name}" Width="36" TextWrapping="NoWrap"/>
              <CheckBox IsChecked="{Binding IsSelected,
                         RelativeSource={RelativeSource AncestorType=ListBoxItem},
                         Mode=TwoWay}"/>
            </StackPanel>
          </Border>
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
  </DockPanel>
</GroupBox>