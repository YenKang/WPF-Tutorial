    <!-- ===== Row 0：IC Select 切換 Primary / L1 / L2 / L3 ===== -->
    <StackPanel Grid.Row="0"
                Orientation="Horizontal"
                Margin="0,0,0,8">
        <!-- 左邊標籤 -->
        <TextBlock Text="IC Select:"
                   FontSize="14"
                   VerticalAlignment="Center"/>

        <!-- 右邊 ComboBox：綁定 ViewModel.AvailableICs + SelectedIC -->
        <ComboBox Width="120"
                  Margin="8,0,0,0"
                  ItemsSource="{Binding AvailableICs}"
                  SelectedItem="{Binding SelectedIC}"
                  HorizontalAlignment="Left">
            <!-- 預設直接用 enum.ToString()：Primary / L1 / L2 / L3 -->
        </ComboBox>
    </StackPanel>