<Grid Width="2560" Height="1440" Background="#0F1A24">
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

        <!-- 主駕 Knob 區 -->
        <Grid Grid.Column="0" Margin="15" RenderTransformOrigin="0.5,0.5">
            <Grid.RenderTransform>
                <RotateTransform x:Name="LeftKnobRotation" Angle="0" />
            </Grid.RenderTransform>
            <Ellipse Width="300" Height="300" Fill="LightGray" Stroke="Gray" StrokeThickness="4" />
            <Ellipse Width="180" Height="180" Fill="Black" />
        </Grid>

        <!-- Dock App 區 -->
        <StackPanel Grid.Column="1" Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="30,0">
            <Button Content="🏠" Width="180" Height="180" Margin="50" FontSize="120" Command="{Binding NavigateToHomeCommand}" />
        </StackPanel>

        <!-- 副駕 Knob 區 -->
        <!-- 可以補 Right Knob -->
    </Grid>

    <!-- 左 Knob Glow（直接加在 Grid，同層級） -->
    <Ellipse x:Name="LeftKnobGlow"
             Width="300" Height="300"
             Fill="#500078FF"
             Stroke="Blue"
             StrokeThickness="2"
             Visibility="Collapsed"/>
</Grid>