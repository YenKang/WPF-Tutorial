<Grid>
  <Grid.RowDefinitions>
    <RowDefinition Height="Auto"/>
    <RowDefinition Height="*"/>
  </Grid.RowDefinitions>

  <!-- 🟦 OSD_EN 摘要表（置於 Header 上方） -->
  <StackPanel Grid.Row="0" Margin="0,0,0,8">
    <TextBlock Text="OSD_EN Summary" FontWeight="Bold" Margin="0,0,0,4"/>

    <Grid>
      <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>  <!-- 索引：30..1 -->
        <RowDefinition Height="Auto"/>  <!-- 數值：1/0 -->
      </Grid.RowDefinitions>

      <!-- 上排：索引 30..1 -->
      <ItemsControl Grid.Row="0" ItemsSource="{Binding Source={StaticResource IconSlotsDesc}}">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <UniformGrid Columns="30"/>
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Border BorderBrush="#E1E4EA" BorderThickness="0,0,1,1" Padding="2" Background="#F7F9FC">
              <TextBlock Text="{Binding IconIndex}" HorizontalAlignment="Center" FontSize="11"/>
            </Border>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>

      <!-- 下排：值 1/0（跟著 IsOsdEnabled 即時變） -->
      <ItemsControl Grid.Row="1" ItemsSource="{Binding Source={StaticResource IconSlotsDesc}}">
        <ItemsControl.ItemsPanel>
          <ItemsPanelTemplate>
            <UniformGrid Columns="30"/>
          </ItemsPanelTemplate>
        </ItemsControl.ItemsPanel>
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Border Padding="2" BorderBrush="#E1E4EA" BorderThickness="0,0,1,1">
              <TextBlock Text="{Binding IsOsdEnabled, Converter={StaticResource Bool01}}"
                         HorizontalAlignment="Center" FontSize="12" FontWeight="SemiBold">
                <TextBlock.Style>
                  <Style TargetType="TextBlock">
                    <Setter Property="Foreground" Value="#666"/>
                    <Style.Triggers>
                      <!-- 勾選時加深顏色 -->
                      <DataTrigger Binding="{Binding IsOsdEnabled}" Value="True">
                        <Setter Property="Foreground" Value="#0F7B0F"/>
                      </DataTrigger>
                    </Style.Triggers>
                  </Style>
                </TextBlock.Style>
              </TextBlock>
            </Border>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </Grid>
  </StackPanel>

  <!-- 你原本的 DataGrid 放這裡 -->
  <DataGrid Grid.Row="1" ...>
    <!-- 既有欄位 -->
  </DataGrid>
</Grid>