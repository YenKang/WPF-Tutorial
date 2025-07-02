<base:NovaPageBase ... >
    <base:NovaPageBase.Resources>
        <Style x:Key="IconButtonStyle" TargetType="Button">
            <Setter Property="Background" Value="Transparent"/>
            <Setter Property="BorderThickness" Value="0"/>
            <Setter Property="Cursor" Value="Hand"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="FontSize" Value="48"/>
            <Setter Property="FontWeight" Value="Bold"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Button">
                        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                            <TextBlock x:Name="Icon" Text="{TemplateBinding Content}" HorizontalAlignment="Center"/>
                            <TextBlock x:Name="Label" Text="{TemplateBinding Tag}" FontSize="24" FontWeight="Bold" HorizontalAlignment="Center"/>
                        </StackPanel>
                        <ControlTemplate.Triggers>
                            <Trigger Property="IsMouseOver" Value="True">
                                <Setter TargetName="Icon" Property="Foreground" Value="#00FFFF"/>
                                <Setter TargetName="Label" Property="Foreground" Value="#00FFFF"/>
                                <Setter TargetName="Icon" Property="RenderTransform">
                                    <Setter.Value>
                                        <ScaleTransform ScaleX="1.1" ScaleY="1.1"/>
                                    </Setter.Value>
                                </Setter>
                                <Setter TargetName="Icon" Property="RenderTransformOrigin" Value="0.5,0.5"/>
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </base:NovaPageBase.Resources>

    <!-- ✅ 以下才是 Grid, Canvas 等畫面 -->
    <Grid>
        <!-- 畫面內容 -->
    </Grid>

</base:NovaPageBase>


<StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="30,0">

    <Button Style="{StaticResource IconButtonStyle}" Width="180" Height="180" Margin="30"
            Content="🏠" Tag="Home"
            Command="{Binding NavigateToHomeCommand}"/>

    <Button Style="{StaticResource IconButtonStyle}" Width="180" Height="180" Margin="30"
            Content="🚗" Tag="Drive"
            Command="{Binding NavigateToDriveCommand}"/>

    <Button Style="{StaticResource IconButtonStyle}" Width="180" Height="180" Margin="30"
            Content="🎵" Tag="Music"
            Command="{Binding NavigateToMusicCommand}"/>

    <Button Style="{StaticResource IconButtonStyle}" Width="180" Height="180" Margin="30"
            Content="❄️" Tag="Climate"
            Command="{Binding NavigateToClimateCommand}"/>

</StackPanel>