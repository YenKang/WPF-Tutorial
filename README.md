<Grid Width="2560" Height="1440" Background="#0F1A24">
    <!-- Grid 本體 (主畫面 + Dock) -->
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="3*" />
            <RowDefinition Height="2*" />
        </Grid.RowDefinitions>

        <ContentControl Grid.Row="0" Content="{Binding CurrentPage}" />

        <Grid Grid.Row="1">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="9*" />
                <ColumnDefinition Width="16*" />
                <ColumnDefinition Width="9*" />
            </Grid.ColumnDefinitions>

            <StackPanel Grid.Column="1" Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
                <Button Content="🏠" Width="180" Height="180" Margin="20" FontSize="120" Command="{Binding NavigateToHomeCommand}" />
            </StackPanel>
        </Grid>
    </Grid>

    <!-- 🔥 新增 Canvas（最外層） -->
    <Canvas IsHitTestVisible="False">
        <!-- 左 Knob 外圈 -->
        <Ellipse x:Name="LeftKnobOuter"
                 Width="371" Height="371"
                 Fill="LightGray"
                 Stroke="Gray"
                 StrokeThickness="4" />

        <!-- 左 Knob 內圈 -->
        <Ellipse x:Name="LeftKnobInner"
                 Width="223" Height="223"
                 Fill="Black" />
    </Canvas>
</Grid>