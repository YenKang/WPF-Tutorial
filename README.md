<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Select Image"
        Width="720" Height="550"
        WindowStartupLocation="CenterOwner"
        ResizeMode="NoResize">

    <DockPanel Margin="12">
        <!-- 標題列（可保留也可移除） -->
        <TextBlock DockPanel.Dock="Top"
                   Text="Select Image"
                   FontWeight="Bold"
                   FontSize="16"
                   Margin="0,0,0,8"/>

        <!-- 圖片牆 -->
        <ScrollViewer VerticalScrollBarVisibility="Auto">
            <ItemsControl x:Name="ic">
                <!-- 版面：水平排列自動換行 -->
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <WrapPanel Orientation="Horizontal"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <!-- 卡片樣板 -->
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Border Margin="6"
                                Padding="8"
                                Width="160" Height="130"
                                Cursor="Hand"
                                BorderThickness="1"
                                MouseLeftButtonUp="OnPick">
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Setter Property="BorderBrush" Value="#DDDDDD"/>
                                    <Setter Property="Background" Value="#FFFFFF"/>
                                    <Style.Triggers>
                                        <!-- 上次選過：淡黃框 -->
                                        <DataTrigger Binding="{Binding IsPreviouslySelected}" Value="True">
                                            <Setter Property="BorderBrush" Value="#FFD966"/>
                                            <Setter Property="Background" Value="#FFFBEA"/>
                                        </DataTrigger>

                                        <!-- 本次點選：藍框（若尚未加 IsCurrentSelected，這段不會生效也不會報錯） -->
                                        <DataTrigger Binding="{Binding IsCurrentSelected}" Value="True">
                                            <Setter Property="BorderBrush" Value="#4A90E2"/>
                                            <Setter Property="Background" Value="#EEF5FF"/>
                                        </DataTrigger>

                                        <!-- 滑鼠移入：深灰框 -->
                                        <Trigger Property="IsMouseOver" Value="True">
                                            <Setter Property="BorderBrush" Value="#999999"/>
                                        </Trigger>
                                    </Style.Triggers>
                                </Style>
                            </Border.Style>

                            <StackPanel>
                                <Image Source="{Binding ImagePath}"
                                       Height="90"
                                       Stretch="UniformToFill"/>
                                <TextBlock Text="{Binding Name}"
                                           Margin="0,6,0,0"
                                           HorizontalAlignment="Center"
                                           TextTrimming="CharacterEllipsis"/>
                            </StackPanel>
                        </Border>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </ScrollViewer>
    </DockPanel>
</Window>