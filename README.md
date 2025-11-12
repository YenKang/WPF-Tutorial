<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        Title="Icon → Image Selection"
        Width="920"
        Height="660"
        WindowStartupLocation="CenterOwner">

    <Window.DataContext>
        <vm:IconToImageMapViewModel/>
    </Window.DataContext>

    <Grid Margin="16">

        <!-- ✅ 直接用 DataGrid Header -->
        <DataGrid ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  HeadersVisibility="Column"
                  GridLinesVisibility="None"
                  RowHeight="44"
                  ColumnHeaderHeight="32"
                  IsReadOnly="False"
                  FontSize="13"
                  ColumnHeaderStyle="{StaticResource {x:Static DataGrid.ColumnHeaderStyleKey}}">

            <DataGrid.Columns>

                <!-- 1️⃣ Icon # -->
                <DataGridTemplateColumn Header="Icon #" Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding IconIndex, StringFormat=Icon #{0}}"
                                       HorizontalAlignment="Center"
                                       VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 2️⃣ Image Selection -->
                <DataGridTemplateColumn Header="Image Selection" Width="250">
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
                <DataGridTemplateColumn Header="SRAM Start Address" Width="160">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding SramStartAddress}"
                                       FontFamily="Consolas"
                                       HorizontalAlignment="Center"
                                       VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 4️⃣ OSD Selection -->
                <DataGridTemplateColumn Header="OSD Selection" Width="140">
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
                <DataGridTemplateColumn Header="OSDx_EN" Width="90">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <CheckBox IsChecked="{Binding IsOsdEnabled, Mode=TwoWay}"
                                      HorizontalAlignment="Center"
                                      VerticalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <!-- 6️⃣ HPos -->
                <DataGridTemplateColumn Header="HPos" Width="90">
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
                <DataGridTemplateColumn Header="VPos" Width="90">
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