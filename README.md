<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Pick an Image" Width="640" Height="520"
        WindowStartupLocation="CenterOwner">

    <!-- 圖片牆（必須 x:Name="ic" 供 .xaml.cs 設 ItemsSource） -->
    <ItemsControl x:Name="ic" Margin="12">
        <ItemsControl.ItemsPanel>
            <ItemsPanelTemplate>
                <WrapPanel />
            </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>

        <!-- 單卡片樣式 -->
        <ItemsControl.ItemTemplate>
            <DataTemplate>
                <Grid>
                    <!-- 外框：本列目前選中（藍） -->
                    <Border Margin="6" CornerRadius="4" BorderThickness="2">
                        <Border.Style>
                            <Style TargetType="Border">
                                <Setter Property="BorderBrush" Value="Transparent"/>
                                <Style.Triggers>
                                    <DataTrigger Binding="{Binding IsCurrentSelected}" Value="True">
                                        <Setter Property="BorderBrush" Value="#2F80ED"/>
                                    </DataTrigger>
                                </Style.Triggers>
                            </Style>
                        </Border.Style>

                        <StackPanel Width="140" Height="140">
                            <Image Source="{Binding ImagePath}" Stretch="UniformToFill" Height="100"/>
                            <TextBlock Text="{Binding Name}" Margin="0,6,0,0"
                                       HorizontalAlignment="Center" TextTrimming="CharacterEllipsis"/>
                        </StackPanel>
                    </Border>

                    <!-- 右上角：其他列使用中（僅標示，不禁用） -->
                    <Border HorizontalAlignment="Right" VerticalAlignment="Top"
                            Margin="0,8,8,0" Padding="4,2"
                            Background="#FFF5B5" CornerRadius="3">
                        <Border.Style>
                            <Style TargetType="Border">
                                <Setter Property="Visibility" Value="Collapsed"/>
                                <Style.Triggers>
                                    <DataTrigger Binding="{Binding IsUsedByOthers}" Value="True">
                                        <Setter Property="Visibility" Value="Visible"/>
                                    </DataTrigger>
                                </Style.Triggers>
                            </Style>
                        </Border.Style>
                        <TextBlock Text="其他列使用中" FontSize="10" Foreground="#7A4B00" FontWeight="Bold"/>
                    </Border>
                </Grid>
            </DataTemplate>
        </ItemsControl.ItemTemplate>

        <!-- 點擊選取（允許重複選） -->
        <ItemsControl.ItemContainerStyle>
            <Style TargetType="ContentPresenter">
                <EventSetter Event="MouseLeftButtonUp" Handler="OnPick"/>
            </Style>
        </ItemsControl.ItemContainerStyle>
    </ItemsControl>
</Window>