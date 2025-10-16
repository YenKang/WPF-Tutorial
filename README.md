<Window x:Class="BistMode.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:sys="clr-namespace:System;assembly=mscorlib"
        Title="BIST UI Bind Test" Width="720" Height="320" Background="#F7F9FC">

    <Window.DataContext>
        <BistModeViewModel/>
    </Window.DataContext>

    <Grid Margin="20">
        <ItemsControl ItemsSource="{Binding ConfigUiRaws}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Border Background="White" BorderBrush="#E0E0E0" BorderThickness="1"
                            CornerRadius="4" Padding="12" Margin="0,0,0,10">
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="240"/> <!-- Title -->
                                <ColumnDefinition Width="*"/>   <!-- D2 -->
                                <ColumnDefinition Width="*"/>   <!-- D1 -->
                                <ColumnDefinition Width="*"/>   <!-- D0 -->
                            </Grid.ColumnDefinitions>

                            <!-- col 1: Title -->
                            <TextBlock Grid.Column="0" Text="{Binding title}" 
                                       FontSize="16" FontWeight="Bold" VerticalAlignment="Center"/>

                            <!-- col 2: D2 -->
                            <ComboBox Grid.Column="1" Width="90" Height="28" HorizontalAlignment="Center">
                                <ComboBox.ItemsSource>
                                    <x:Array Type="{x:Type sys:Int32}">
                                        <sys:Int32>0</sys:Int32>
                                        <sys:Int32>1</sys:Int32>
                                        <sys:Int32>2</sys:Int32>
                                        <sys:Int32>3</sys:Int32>
                                    </x:Array>
                                </ComboBox.ItemsSource>
                                <ComboBox.SelectedIndex>
                                    <Binding Path="fields[D2].@default"/>
                                </ComboBox.SelectedIndex>
                            </ComboBox>

                            <!-- col 3: D1 -->
                            <ComboBox Grid.Column="2" Width="90" Height="28" HorizontalAlignment="Center">
                                <ComboBox.ItemsSource>
                                    <x:Array Type="{x:Type sys:Int32}">
                                        <sys:Int32>0</sys:Int32>
                                        <sys:Int32>1</sys:Int32>
                                        <sys:Int32>2</sys:Int32>
                                        <sys:Int32>3</sys:Int32>
                                        <sys:Int32>4</sys:Int32>
                                        <sys:Int32>5</sys:Int32>
                                        <sys:Int32>6</sys:Int32>
                                        <sys:Int32>7</sys:Int32>
                                        <sys:Int32>8</sys:Int32>
                                        <sys:Int32>9</sys:Int32>
                                        <sys:Int32>10</sys:Int32>
                                        <sys:Int32>11</sys:Int32>
                                        <sys:Int32>12</sys:Int32>
                                        <sys:Int32>13</sys:Int32>
                                        <sys:Int32>14</sys:Int32>
                                        <sys:Int32>15</sys:Int32>
                                    </x:Array>
                                </ComboBox.ItemsSource>
                                <ComboBox.SelectedIndex>
                                    <Binding Path="fields[D1].@default"/>
                                </ComboBox.SelectedIndex>
                            </ComboBox>

                            <!-- col 4: D0 -->
                            <ComboBox Grid.Column="3" Width="90" Height="28" HorizontalAlignment="Center">
                                <ComboBox.ItemsSource>
                                    <x:Array Type="{x:Type sys:Int32}">
                                        <sys:Int32>0</sys:Int32>
                                        <sys:Int32>1</sys:Int32>
                                        <sys:Int32>2</sys:Int32>
                                        <sys:Int32>3</sys:Int32>
                                        <sys:Int32>4</sys:Int32>
                                        <sys:Int32>5</sys:Int32>
                                        <sys:Int32>6</sys:Int32>
                                        <sys:Int32>7</sys:Int32>
                                        <sys:Int32>8</sys:Int32>
                                        <sys:Int32>9</sys:Int32>
                                        <sys:Int32>10</sys:Int32>
                                        <sys:Int32>11</sys:Int32>
                                        <sys:Int32>12</sys:Int32>
                                        <sys:Int32>13</sys:Int32>
                                        <sys:Int32>14</sys:Int32>
                                        <sys:Int32>15</sys:Int32>
                                    </x:Array>
                                </ComboBox.ItemsSource>
                                <ComboBox.SelectedIndex>
                                    <Binding Path="fields[D0].@default"/>
                                </ComboBox.SelectedIndex>
                            </ComboBox>
                        </Grid>
                    </Border>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </Grid>
</Window>