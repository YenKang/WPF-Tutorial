<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        mc:Ignorable="d"
        Title="Icon → Image Selection"
        Width="1300" Height="820"
        WindowStartupLocation="CenterScreen"
        Background="#F9FAFB">

    <Window.DataContext>
        <vm:IconToImageMapViewModel/>
    </Window.DataContext>

    <Window.Resources>
        <!-- Boolean → 0/1 -->
        <BooleanToVisibilityConverter x:Key="BoolToVis"/>
        <vm:BoolTo01Converter x:Key="Bool01"/>
    </Window.Resources>

    <DockPanel Margin="15">

        <!-- 🟩 Summary 區：OSD_EN Summary -->
        <StackPanel DockPanel.Dock="Top" Margin="0,0,0,12">
            <TextBlock Text="OSD_EN Summary" 
                       FontSize="14" 
                       FontWeight="Bold"
                       Foreground="#333"
                       Margin="0,0,0,4"/>
            
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>

                <!-- 標題列 30→1 -->
                <ItemsControl Grid.Row="0" ItemsSource="{Binding IconSlots}">
                    <ItemsControl.ItemsPanel>
                        <ItemsPanelTemplate>
                            <UniformGrid Columns="30"/>
                        </ItemsPanelTemplate>
                    </ItemsControl.ItemsPanel>
                    <ItemsControl.ItemTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding IconIndex}"
                                       HorizontalAlignment="Center"
                                       FontSize="12"
                                       Foreground="#5A6472"
                                       FontWeight="SemiBold"/>
                        </DataTemplate>
                    </ItemsControl.ItemTemplate>
                </ItemsControl>

                <!-- 顏色狀態列 -->
                <ItemsControl Grid.Row="1" ItemsSource="{Binding IconSlots}">
                    <ItemsControl.ItemsPanel>
                        <ItemsPanelTemplate>
                            <UniformGrid Columns="30"/>
                        </ItemsPanelTemplate>
                    </ItemsControl.ItemsPanel>

                    <ItemsControl.ItemTemplate>
                        <DataTemplate>
                            <Border x:Name="cell"
                                    Padding="3" Margin="1"
                                    CornerRadius="3"
                                    BorderThickness="1"
                                    BorderBrush="#D9DEE8"
                                    Background="#F7F9FC">
                                <TextBlock Text="{Binding IsOsdEnabled, Converter={StaticResource Bool01}}"
                                           FontSize="13"
                                           FontWeight="SemiBold"
                                           HorizontalAlignment="Center"
                                           VerticalAlignment="Center"/>
                            </Border>

                            <!-- 🎨 顏色變化 -->
                            <DataTemplate.Triggers>
                                <!-- 勾選 (1) 綠底白字 -->
                                <DataTrigger Binding="{Binding IsOsdEnabled}" Value="True">
                                    <Setter TargetName="cell" Property="Background" Value="#2E7D32"/>
                                    <Setter TargetName="cell" Property="BorderBrush" Value="#2E7D32"/>
                                    <Setter TargetName="cell" Property="TextBlock.Foreground" Value="White"/>
                                </DataTrigger>

                                <!-- 未勾 (0) 灰底灰字 -->
                                <DataTrigger Binding="{Binding IsOsdEnabled}" Value="False">
                                    <Setter TargetName="cell" Property="Background" Value="#ECEFF1"/>
                                    <Setter TargetName="cell" Property="BorderBrush" Value="#D0D5DA"/>
                                    <Setter TargetName="cell" Property="TextBlock.Foreground" Value="#5A6472"/>
                                </DataTrigger>
                            </DataTemplate.Triggers>
                        </DataTemplate>
                    </ItemsControl.ItemTemplate>
                </ItemsControl>
            </Grid>
        </StackPanel>

        <!-- 🟦 主表格 -->
        <DataGrid ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  HeadersVisibility="Column"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  CanUserResizeRows="False"
                  RowHeaderWidth="0"
                  GridLinesVisibility="None"
                  AlternatingRowBackground="#F4F7FB"
                  AlternationCount="2"
                  RowHeight="32"
                  Margin="0,0,0,10">

            <!-- 📘 Column 定義 -->
            <DataGrid.Columns>
                <DataGridTextColumn Header="ICON #" Binding="{Binding IconIndex}" IsReadOnly="True" Width="90"/>

                <!-- Image Selection -->
                <DataGridTemplateColumn Header="IMAGE SELECTION" Width="180">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="{Binding SelectedImageName}" 
                                    Command="{Binding DataContext.OpenPickerCommand, RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                    CommandParameter="{Binding}"
                                    Padding="4" Margin="2" Height="24"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- SRAM Start Address -->
                <DataGridTextColumn Header="SRAM START ADDRESS" Binding="{Binding SramStartAddress}" Width="160"/>

                <!-- OSD # -->
                <DataGridTextColumn Header="OSD #" Binding="{Binding OsdIndex}" Width="100"/>

                <!-- OSD Icon Selection -->
                <DataGridTemplateColumn Header="OSD ICON SELECTION" Width="180">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <ComboBox ItemsSource="{Binding DataContext.SramButtonList, RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                      SelectedItem="{Binding SelectedOsdImage, Mode=TwoWay}"
                                      DisplayMemberPath="Name"
                                      Width="160" Height="24"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- OSDx_EN -->
                <DataGridCheckBoxColumn Header="OSDXEN"
                                        Binding="{Binding IsOsdEnabled, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                        Width="80"/>

                <!-- TTALEN -->
                <DataGridCheckBoxColumn Header="TTALEEN"
                                        Binding="{Binding IsTtaleEnabled, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                        Width="80"/>

                <!-- HPos -->
                <DataGridTextColumn Header="HPOS" Binding="{Binding HPos}" Width="80"/>

                <!-- VPos -->
                <DataGridTextColumn Header="VPOS" Binding="{Binding VPos}" Width="80"/>
            </DataGrid.Columns>
        </DataGrid>

        <!-- 🟠 底部按鈕 -->
        <StackPanel DockPanel.Dock="Bottom" Orientation="Horizontal" HorizontalAlignment="Right" Margin="0,10,0,0">
            <Button Content="OK" Width="90" Height="30" Margin="4" IsDefault="True" Click="Ok_Click"/>
            <Button Content="Cancel" Width="90" Height="30" Margin="4" IsCancel="True"/>
        </StackPanel>
    </DockPanel>
</Window>