<Window x:Class="OSDIconFlashMap.View.ImagePickerWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Pick an Image" Width="640" Height="520"
        WindowStartupLocation="CenterOwner">

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <!-- 原本的 ScrollViewer 整段搬到 Row 0 -->
        <ScrollViewer Grid.Row="0"
                      VerticalScrollBarVisibility="Auto"
                      HorizontalScrollBarVisibility="Disabled">
            <ItemsControl x:Name="it" Margin="12">
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <WrapPanel />
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Grid>
                            <!-- 你原本 DataTemplate 裡的內容 그대로 保留 -->
                        </Grid>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>

                <ItemsControl.ItemContainerStyle>
                    <Style TargetType="ContentPresenter">
                        <EventSetter Event="MouseLeftButtonUp" Handler="OnPick"/>
                    </Style>
                </ItemsControl.ItemContainerStyle>
            </ItemsControl>
        </ScrollViewer>

        <!-- 新增：右下角 Clear / Confirm -->
        <StackPanel Grid.Row="1"
                    Orientation="Horizontal"
                    HorizontalAlignment="Right"
                    Margin="8">
            <Button Content="Clear"
                    Width="90"
                    Margin="0,0,8,0"
                    Click="Clear_Click"/>
            <Button Content="Confirm"
                    Width="90"
                    Click="Confirm_Click"/>
        </StackPanel>
    </Grid>
</Window>
