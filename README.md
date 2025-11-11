<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Select Image"
        Width="720" Height="550"
        WindowStartupLocation="CenterOwner"
        ResizeMode="NoResize">
    <Window.Resources>
        <BooleanToVisibilityConverter x:Key="BoolToVisibilityConverter"/>
    </Window.Resources>

    <DockPanel Margin="12">
        <TextBlock DockPanel.Dock="Top"
                   Text="Select Image"
                   FontWeight="Bold"
                   FontSize="16"
                   Margin="0,0,0,8"/>

        <ScrollViewer VerticalScrollBarVisibility="Auto">
            <ItemsControl x:Name="ic">
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <WrapPanel Orientation="Horizontal"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

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
                                        <!-- 上次選過：淡黃底 -->
                                        <DataTrigger Binding="{Binding IsPreviouslySelected}"
                                                     Value="True">
                                            <Setter Property="BorderBrush" Value="#FFD966"/>
                                            <Setter Property="Background" Value="#FFFBEA"/>
                                        </DataTrigger>

                                        <!-- 本次點選：藍框 -->
                                        <DataTrigger Binding="{Binding IsCurrentSelected}"
                                                     Value="True">
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

                            <!-- 卡片內容與「已被使用」角標 -->
                            <Grid>
                                <StackPanel>
                                    <Image Source="{Binding ImagePath}"
                                           Height="90"
                                           Stretch="UniformToFill"/>
                                    <TextBlock Text="{Binding Name}"
                                               Margin="0,6,0,0"
                                               HorizontalAlignment="Center"
                                               TextTrimming="CharacterEllipsis"/>
                                </StackPanel>

                                <!-- 右上角：其他列已使用 -->
                                <Border Background="#FFE8B5"
                                        CornerRadius="3"
                                        HorizontalAlignment="Right"
                                        VerticalAlignment="Top"
                                        Margin="0,0,4,0"
                                        Padding="4,1"
                                        Visibility="Collapsed">
                                    <Border.Visibility>
                                        <Binding Path="IsUsedByOthers"
                                                 Converter="{StaticResource BoolToVisibilityConverter}"/>
                                    </Border.Visibility>
                                    <TextBlock Text="已被使用"
                                               FontSize="10"
                                               Foreground="#7A4B00"/>
                                </Border>
                            </Grid>
                        </Border>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </ScrollViewer>
    </DockPanel>
</Window>