<ListView ItemsSource="{Binding Patterns}"
          BorderThickness="0"
          Background="Transparent"
          ScrollViewer.HorizontalScrollBarVisibility="Disabled"
          ScrollViewer.VerticalScrollBarVisibility="Auto">
    <ListView.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel Width="800" Orientation="Horizontal"/>
        </ItemsPanelTemplate>
    </ListView.ItemsPanel>

    <ListView.ItemTemplate>
        <DataTemplate>
            <!-- 每一個 Item 外面用 Border 當格子 -->
            <Border Width="150" Height="130" Margin="4"
                    BorderBrush="Gray" BorderThickness="1">
                <StackPanel>
                    <Image Source="{Binding Icon}" 
                           Width="120" Height="70" 
                           Stretch="UniformToFill" 
                           HorizontalAlignment="Center"/>
                    <TextBlock Text="{Binding Name}" 
                               Width="120"
                               TextWrapping="Wrap"
                               TextAlignment="Center"
                               Margin="0,6,0,0"
                               HorizontalAlignment="Center"/>
                </StackPanel>
            </Border>
        </DataTemplate>
    </ListView.ItemTemplate>

    <!-- (可選) 取消藍色選取背景 -->
    <ListView.ItemContainerStyle>
        <Style TargetType="ListViewItem">
            <Setter Property="BorderThickness" Value="0"/>
            <Setter Property="Padding" Value="0"/>
            <Setter Property="Margin" Value="0"/>
            <Setter Property="HorizontalContentAlignment" Value="Stretch"/>
            <Setter Property="VerticalContentAlignment" Value="Stretch"/>
            <Setter Property="Background" Value="Transparent"/>
        </Style>
    </ListView.ItemContainerStyle>
</ListView>