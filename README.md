<Window x:Class="OSDIconFlashMap.View.IconToImageMapWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:OSDIconFlashMap.ViewModel"
        xmlns:model="clr-namespace:OSDIconFlashMap.Model"
        Title="Icon → Image Selection"
        Width="900"
        Height="640"
        WindowStartupLocation="CenterOwner">

    <Window.DataContext>
        <!-- 預設 VM：若外部沒注入會自動建立 -->
        <vm:IconToImageMapViewModel/>
    </Window.DataContext>

    <Grid Margin="16">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- === Header 標題列 === -->
        <DockPanel Grid.Row="0" Margin="0,0,0,8">
            <TextBlock Text="Icon #" Width="120" FontWeight="Bold" VerticalAlignment="Center"/>
            <TextBlock Text="Image Selection" Width="250" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="SRAM Start Address" Width="160" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="OSD Selection" Width="140" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="OSDx_EN" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="HPos" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
            <TextBlock Text="VPos" Width="90" FontWeight="Bold" Margin="16,0,0,0" VerticalAlignment="Center"/>
        </DockPanel>

        <!-- === 主表格 === -->
        <DataGrid Grid.Row="1"
                  ItemsSource="{Binding IconSlots}"
                  AutoGenerateColumns="False"
                  CanUserAddRows="False"
                  CanUserDeleteRows="False"
                  HeadersVisibility="None"
                  GridLinesVisibility="None"
                  RowHeight="44"
                  IsReadOnly="False">

            <DataGrid.Columns>
                <!-- 1️⃣ Icon #N -->
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
                <DataGridCheckBoxColumn Width="90"
                                        Binding="{Binding IsOsdEnabled, Mode=TwoWay}"
                                        EditingElementStyle="{x:Null}"/>

                <!-- 6️⃣ HPos -->
                <DataGridTextColumn Width="90"
                                    Binding="{Binding HPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>

                <!-- 7️⃣ VPos -->
                <DataGridTextColumn Width="90"
                                    Binding="{Binding VPos, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>