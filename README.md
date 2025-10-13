<Grid.Resources>
  <SolidColorBrush x:Key="CardBg" Color="#FFFFFF"/>
  <SolidColorBrush x:Key="Border1" Color="#E1E4EA"/>
  <SolidColorBrush x:Key="HeaderBg" Color="#F0F3F8"/>
  <SolidColorBrush x:Key="HeaderFg" Color="#2A2F3A"/>

  <Style x:Key="SectionGroupBox" TargetType="GroupBox">
    <Setter Property="Margin" Value="0,0,0,12"/>
    <Setter Property="Padding" Value="0"/>
    <Setter Property="Template">
      <Setter.Value>
        <ControlTemplate TargetType="GroupBox">
          <Border Background="{StaticResource CardBg}"
                  BorderBrush="{StaticResource Border1}"
                  BorderThickness="1"
                  CornerRadius="10">
            <Grid>
              <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
              </Grid.RowDefinitions>

              <Border Background="{StaticResource HeaderBg}"
                      CornerRadius="10,10,0,0"
                      Padding="10,6"
                      BorderBrush="{StaticResource Border1}"
                      BorderThickness="0,0,0,1">
                <ContentPresenter ContentSource="Header"
                                  TextElement.FontWeight="SemiBold"
                                  TextElement.Foreground="{StaticResource HeaderFg}"/>
              </Border>

              <Border Grid.Row="1" Padding="12">
                <ContentPresenter/>
              </Border>
            </Grid>
          </Border>
        </ControlTemplate>
      </Setter.Value>
    </Setter>
  </Style>
</Grid.Resources>
