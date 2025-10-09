<DataTemplate>
    <Border CornerRadius="3" BorderThickness="1" Background="Transparent">
        <!-- 垂直排版：上面 CheckBox、下面 Button -->
        <StackPanel Orientation="Vertical" HorizontalAlignment="Center" Margin="2">
            <!-- 上：原本的 CheckBox（顯示 P{Index}，選取樣式保持） -->
            <CheckBox
                HorizontalContentAlignment="Center"
                VerticalContentAlignment="Center"
                FontSize="12"
                Content="{Binding Index, StringFormat=P{0}}"
                IsChecked="{Binding IsSelected,
                            RelativeSource={RelativeSource AncestorType=ListBoxItem},
                            Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
                <CheckBox.Style>
                    <Style TargetType="CheckBox">
                        <Setter Property="Background" Value="WhiteSmoke"/>
                        <Setter Property="BorderThickness" Value="1"/>
                        <Setter Property="BorderBrush" Value="Black"/>
                        <Style.Triggers>
                            <Trigger Property="IsChecked" Value="True">
                                <Setter Property="Foreground" Value="White"/>
                                <Setter Property="Background" Value="Yellow"/>
                            </Trigger>
                        </Style.Triggers>
                    </Style>
                </CheckBox.Style>
            </CheckBox>

            <!-- 下：新增的 Button -->
            <Button Content="Set"
                    Margin="0,4,0,0"
                    Padding="2,0"
                    FontSize="10"
                    HorizontalAlignment="Center"
                    Command="{Binding DataContext.ApplyPatternCommand,
                                      RelativeSource={RelativeSource AncestorType=ListBox}}"
                    CommandParameter="{Binding}" />
        </StackPanel>
    </Border>
</DataTemplate>