<GroupBox Header="CSP Config" Margin="8">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>   <!-- 表頭 -->
            <RowDefinition Height="*"/>      <!-- 內容 -->
        </Grid.RowDefinitions>

        <!-- ========== 表頭 (CSP Register / Value / UI) ========== -->
        <Grid Grid.Row="0" Margin="0,0,0,4">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="150"/>   <!-- CSP Register -->
                <ColumnDefinition Width="120"/>   <!-- Value -->
                <ColumnDefinition Width="*"/>     <!-- UI -->
            </Grid.ColumnDefinitions>

            <Border Grid.Column="0" BorderBrush="Gray" BorderThickness="0,0,1,1">
                <TextBlock Text="CSP Register"
                           HorizontalAlignment="Center"
                           VerticalAlignment="Center"
                           FontWeight="Bold"
                           Margin="4,2"/>
            </Border>

            <Border Grid.Column="1" BorderBrush="Gray" BorderThickness="0,0,1,1">
                <TextBlock Text="Value"
                           HorizontalAlignment="Center"
                           VerticalAlignment="Center"
                           FontWeight="Bold"
                           Margin="4,2"/>
            </Border>

            <Border Grid.Column="2" BorderBrush="Gray" BorderThickness="0,0,0,1">
                <TextBlock Text="UI"
                           HorizontalAlignment="Center"
                           VerticalAlignment="Center"
                           FontWeight="Bold"
                           Margin="4,2"/>
            </Border>
        </Grid>

        <!-- ========== 內容：每個 CspSlotModel 一塊 datasheet 表格 ========== -->
        <ItemsControl Grid.Row="1"
                      ItemsSource="{Binding CspSlots}">
            <ItemsControl.ItemsPanel>
                <ItemsPanelTemplate>
                    <StackPanel Orientation="Vertical"/>
                </ItemsPanelTemplate>
            </ItemsControl.ItemsPanel>

            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Border BorderBrush="Gray" BorderThickness="1" Margin="0,0,0,4" Padding="4">
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <!-- 左邊 Register 名稱 -->
                                <ColumnDefinition Width="150"/>
                                <!-- 中間 Value -->
                                <ColumnDefinition Width="120"/>
                                <!-- 中間空白 -->
                                <ColumnDefinition Width="20"/>
                                <!-- 右邊 Preview（合併儲存格） -->
                                <ColumnDefinition Width="*"/>
                            </Grid.ColumnDefinitions>

                            <Grid.RowDefinitions>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                                <RowDefinition Height="Auto"/>
                            </Grid.RowDefinitions>

                            <!-- ===== Row 1: CSP_ALPHA_EN ===== -->
                            <TextBlock Grid.Row="0" Grid.Column="0"
                                       Text="CSP_ALPHA_EN"
                                       Margin="2"
                                       VerticalAlignment="Center"/>

                            <CheckBox  Grid.Row="0" Grid.Column="1"
                                       Margin="2"
                                       Content="Enable"
                                       IsChecked="{Binding AlphaEnable, Mode=TwoWay}"/>

                            <!-- ===== Row 2: CSP_ALPHA_SEL[5:0] ===== -->
                            <TextBlock Grid.Row="1" Grid.Column="0"
                                       Text="CSP_ALPHA_SEL[5:0]"
                                       Margin="2"
                                       VerticalAlignment="Center"/>

                            <xctk:IntegerUpDown Grid.Row="1" Grid.Column="1"
                                                Margin="2"
                                                Minimum="0"
                                                Maximum="63"
                                                Value="{Binding AlphaSel, Mode=TwoWay}"/>

                            <!-- ===== Row 3: CSP_R[7:0] ===== -->
                            <TextBlock Grid.Row="2" Grid.Column="0"
                                       Text="CSP_R[7:0]"
                                       Margin="2"
                                       VerticalAlignment="Center"/>

                            <xctk:IntegerUpDown Grid.Row="2" Grid.Column="1"
                                                Margin="2"
                                                Minimum="0"
                                                Maximum="255"
                                                Value="{Binding R, Mode=TwoWay}"/>

                            <!-- ===== Row 4: CSP_G[7:0] ===== -->
                            <TextBlock Grid.Row="3" Grid.Column="0"
                                       Text="CSP_G[7:0]"
                                       Margin="2"
                                       VerticalAlignment="Center"/>

                            <xctk:IntegerUpDown Grid.Row="3" Grid.Column="1"
                                                Margin="2"
                                                Minimum="0"
                                                Maximum="255"
                                                Value="{Binding G, Mode=TwoWay}"/>

                            <!-- ===== Row 5: CSP_B[7:0] ===== -->
                            <TextBlock Grid.Row="4" Grid.Column="0"
                                       Text="CSP_B[7:0]"
                                       Margin="2"
                                       VerticalAlignment="Center"/>

                            <xctk:IntegerUpDown Grid.Row="4" Grid.Column="1"
                                                Margin="2"
                                                Minimum="0"
                                                Maximum="255"
                                                Value="{Binding B, Mode=TwoWay}"/>

                            <!-- ===== 右邊合併儲存格 Preview（RowSpan=5） ===== -->
                            <Border Grid.Column="3"
                                    Grid.Row="0"
                                    Grid.RowSpan="5"
                                    Margin="12,2,2,2"
                                    BorderBrush="DarkGreen"
                                    BorderThickness="2"
                                    SnapsToDevicePixels="True">
                                <Rectangle Fill="{Binding PreviewBrush}"
                                           Opacity="{Binding PreviewOpacity}"/>
                            </Border>
                        </Grid>
                    </Border>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </Grid>
</GroupBox>