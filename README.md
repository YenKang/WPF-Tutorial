<GroupBox Header="Pattern En" Margin="12">
    <ListBox ItemsSource="{Binding Patterns}"
             SelectedItem="{Binding SelectedPattern, Mode=TwoWay}"
             SelectionMode="Single"
             BorderThickness="0"
             ScrollViewer.HorizontalScrollBarVisibility="Disabled"
             HorizontalContentAlignment="Left">
        <!-- 可換行 -->
        <ListBox.ItemsPanel>
            <ItemsPanelTemplate>
                <WrapPanel />
            </ItemsPanelTemplate>
        </ListBox.ItemsPanel>

        <!-- 壓縮容器間距，做成「P1 [ ]」細長型 -->
        <ListBox.ItemContainerStyle>
            <Style TargetType="ListBoxItem">
                <Setter Property="Padding" Value="0"/>
                <Setter Property="Margin" Value="4,2"/>
            </Style>
        </ListBox.ItemContainerStyle>

        <!-- 單向 item：文字 + 小勾框 -->
        <ListBox.ItemTemplate>
            <DataTemplate>
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="{Binding Name}" Margin="0,0,6,0" />
                    <!-- 勾選狀態 = 是否被選取；可勾選或取消 -->
                    <CheckBox Width="14" Height="14"
                              VerticalAlignment="Center"
                              IsChecked="{Binding IsSelected,
                                  RelativeSource={RelativeSource AncestorType=ListBoxItem},
                                  Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
                </StackPanel>
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
</GroupBox>