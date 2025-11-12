<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        Title="Icon → Image Selection"
        Width="900"
        Height="640"
        WindowStartupLocation="CenterOwner">

    <Window.DataContext>
        <vm:IconToImageMapViewModel/>
    </Window.DataContext>

    <Grid Margin="16">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- 標題列 -->
        <DockPanel Grid.Row="0" Margin="0,0,0,8">
            <TextBlock Text="Icon #" Width="120" FontWeight="Bold" VerticalAlignment="Center"/>
            <TextBlock Text="Image Selection" Width="250" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="SRAM Start Address" Width="160" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="OSD Selection" Width="140" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="OSDx_EN" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="HPos" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="VPos" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
        </DockPanel>

        <!-- 主內容 DataGrid -->
        <DataGrid Grid.Row="1"
                  ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  HeadersVisibility="None"
                  GridLinesVisibility="None"
                  RowHeight="44">

            <DataGrid.Columns>

                <!-- 1️⃣ Icon # -->
                <DataGridTemplateColumn Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                                       VerticalAlignment="Center"
                                       HorizontalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 2️⃣ Image Selection -->
                <DataGridTemplateColumn Width="250">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Button Content="{Binding SelectedImageName}"
                                    HorizontalAlignment="Stretch"
                                    Padding="10,6"
                                    Click="OpenPicker_Click"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 3️⃣ SRAM Start Address -->
                <DataGridTemplateColumn Width="160">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding SramStartAddress}"
                                       FontFamily="Consolas"
                                       VerticalAlignment="Center"
                                       HorizontalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 4️⃣ OSD Selection -->
                <DataGridTemplateColumn Width="140">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <ComboBox Width="120"
                                      SelectedValuePath="."
                                      SelectedValue="{Binding OsdTargetIndex, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
                                <ComboBox.ItemsSource>
                                    <Binding Path="DataContext.OsdOptions"
                                             RelativeSource="{RelativeSource AncestorType=Window}"/>
                                </ComboBox.ItemsSource>
                            </ComboBox>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 5️⃣ OSDx_EN -->
                <DataGridTemplateColumn Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <CheckBox IsChecked="{Binding IsOsdEnabled, Mode=TwoWay}"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 6️⃣ HPos -->
                <DataGridTemplateColumn Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBox Text="{Binding HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                     HorizontalAlignment="Center"
                                     VerticalAlignment="Center"
                                     Width="70"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 7️⃣ VPos -->
                <DataGridTemplateColumn Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBox Text="{Binding VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                                     HorizontalAlignment="Center"
                                     VerticalAlignment="Center"
                                     Width="70"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>