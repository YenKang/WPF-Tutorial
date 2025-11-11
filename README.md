<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Select Image"
        Width="720"
        Height="550"
        WindowStartupLocation="CenterOwner"
        ResizeMode="NoResize">

    <DockPanel Margin="12">

        <!-- 標題列 -->
        <TextBlock DockPanel.Dock="Top"
                   Text="Select Image"
                   FontWeight="Bold"
                   FontSize="16"
                   Margin="0,0,0,8"/>

        <!-- 主要圖片牆 -->
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
                                Width="160"
                                Height="130"
                                BorderThickness="1"
                                Cursor="Hand"
                                MouseLeftButtonUp="OnPick">

                            <!-- 邊框樣式控制 -->
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Setter Property="BorderBrush" Value="#DDDDDD"/>
                                    <Setter Property="Background" Value="#FFFFFF"/>

                                    <Style.Triggers>

                                        <!-- (1) 上次選過：黃框、淡黃底 -->
                                        <DataTrigger Binding="{Binding IsPreviouslySelected}" Value="True">
                                            <Setter Property="BorderBrush" Value="#FFD966"/>
                                            <Setter Property="Background" Value="#FFFBEA"/>
                                        </DataTrigger>

                                        <!-- (2) 目前選中：藍框 -->
                                        <DataTrigger Binding="{Binding IsCurrentSelected}" Value="True">
                                            <Setter Property="BorderBrush" Value="#4A90E2"/>
                                            <Setter Property="Background" Value="#EEF5FF"/>
                                        </DataTrigger>

                                        <!-- (3) 滑鼠移入：深灰框 -->
                                        <Trigger Property="IsMouseOver" Value="True">
                                            <Setter Property="BorderBrush" Value="#999999"/>
                                        </Trigger>
                                    </Style.Triggers>
                                </Style>
                            </Border.Style>

                            <!-- 內容區 -->
                            <Grid>
                                <StackPanel>
                                    <Image Source="{Binding ImagePath}"
                                           Height="90"
                                           Stretch="UniformToFill"
                                           Margin="0,0,0,4"/>
                                    <TextBlock Text="{Binding Name}"
                                               FontSize="13"
                                               HorizontalAlignment="Center"
                                               TextTrimming="CharacterEllipsis"/>
                                </StackPanel>

                                <!-- 右上角：已被其他列使用 -->
                                <Border HorizontalAlignment="Right"
                                        VerticalAlignment="Top"
                                        Margin="0,0,4,0"
                                        Padding="4,1"
                                        CornerRadius="3">
                                    <Border.Style>
                                        <Style TargetType="Border">
                                            <Setter Property="Visibility" Value="Collapsed"/>
                                            <Setter Property="Background" Value="#FFE8B5"/>
                                            <Style.Triggers>
                                                <DataTrigger Binding="{Binding IsUsedByOthers}" Value="True">
                                                    <Setter Property="Visibility" Value="Visible"/>
                                                </DataTrigger>
                                            </Style.Triggers>
                                        </Style>
                                    </Border.Style>
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