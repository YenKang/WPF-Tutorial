<Grid>
    <Grid.RowDefinitions>
        <RowDefinition />               <!-- 上面：TabControl 區域 -->
        <RowDefinition Height="Auto"/>  <!-- 下面：按鈕列 -->
    </Grid.RowDefinitions>

    <!-- ① TabControl：兩個 Tab 的內容 -->
    <TabControl Grid.Row="0">
        <TabItem Header="Main Setting">
            <!-- Main Setting 的內容 -->
        </TabItem>

        <TabItem Header="ASIL">
            <!-- ASIL 的內容 -->
        </TabItem>
    </TabControl>

    <!-- 共同按鈕列：無論在哪個 Tab 都看得到 -->
    <StackPanel Grid.Row="1"
                Orientation="Horizontal"
                HorizontalAlignment="Right"
                Margin="0,8,0,0">
        <Button Content="Write Reg Table"
                Width="100"
                Margin="0,0,8,0"
                Command="{Binding WriteRegCommand}" />

        <Button Content="Export"
                Width="80"
                Margin="0,0,8,0"
                Command="{Binding ExportCommand}" />

        <Button Content="Cancel"
                Width="80"
                Command="{Binding CancelCommand}" />
    </StackPanel>
</Grid>
