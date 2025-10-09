<GroupBox Header="Pattern En" Margin="12">
  <ListBox ItemsSource="{Binding Patterns}"
           SelectedItem="{Binding SelectedPattern, Mode=TwoWay}"
           SelectionMode="Single"
           BorderThickness="0"
           ScrollViewer.HorizontalScrollBarVisibility="Disabled">

    <!-- 自動換行，並可調每格尺寸 -->
    <ListBox.ItemsPanel>
      <ItemsPanelTemplate>
        <WrapPanel ItemWidth="76" ItemHeight="24"/>
      </ItemsPanelTemplate>
    </ListBox.ItemsPanel>

    <ListBox.ItemContainerStyle>
      <Style TargetType="ListBoxItem">
        <Setter Property="Padding" Value="0"/>
        <Setter Property="Margin" Value="2"/>
        <Setter Property="HorizontalContentAlignment" Value="Stretch"/>
        <Setter Property="VerticalContentAlignment" Value="Stretch"/>
      </Style>
    </ListBox.ItemContainerStyle>

    <!-- 單一元件：CheckBox，Content 顯示 Index（P{Index}） -->
    <ListBox.ItemTemplate>
      <DataTemplate>
        <Border CornerRadius="3" BorderBrush="#666" BorderThickness="1" Background="Transparent">
          <CheckBox
              HorizontalContentAlignment="Center"
              VerticalContentAlignment="Center"
              Padding="4,0"
              FontSize="12"
              Content="{Binding Index, StringFormat=P{0}}"
              IsChecked="{Binding IsSelected,
                          RelativeSource={RelativeSource AncestorType=ListBoxItem},
                          Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
            <!-- 勾選時讓底色與邊框更明顯 -->
            <CheckBox.Style>
              <Style TargetType="CheckBox">
                <Setter Property="Background" Value="Transparent"/>
                <Setter Property="BorderThickness" Value="0"/>
                <Style.Triggers>
                  <Trigger Property="IsChecked" Value="True">
                    <Setter Property="Foreground" Value="White"/>
                    <Setter Property="Background" Value="#2196F3"/>
                  </Trigger>
                </Style.Triggers>
              </Style>
            </CheckBox.Style>
          </CheckBox>
        </Border>
      </DataTemplate>
    </ListBox.ItemTemplate>
  </ListBox>
</GroupBox>