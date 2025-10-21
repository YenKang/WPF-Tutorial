<!-- Auto-Run Config -->
<GroupBox Header="Auto-Run Config"
         Visibility="{Binding IsAutoRunConfigVisible, Converter={StaticResource BoolToVisibilityConverter}}"
         Margin="0,12,0,0">
  <StackPanel Margin="12" DataContext="{Binding AutoRunVM}">
    
    <!-- Pattern Total -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Pattern Total:" VerticalAlignment="Center" Margin="0,0,8,0"/>
      <ComboBox Width="80"
                ItemsSource="{Binding TotalOptions}"
                SelectedItem="{Binding Total, Mode=TwoWay}"/>
    </StackPanel>

    <!-- Orders: 依 Total 動態顯示 -->
    <ItemsControl ItemsSource="{Binding Orders}">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <StackPanel Orientation="Horizontal" Margin="0,2,0,2">
            <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ItemsControl}, 
                                      Path=TemplatedParent.DataContext.(local:AutoRunVM.Title)}" Visibility="Collapsed"/>
            <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ContentPresenter}, 
                                      Path=ContentPresenter.ContentTemplateSelector}" Visibility="Collapsed"/>
            <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ItemsControl}, Path=Items.CurrentPosition}" Visibility="Collapsed"/>
            <TextBlock Text="{Binding}" Visibility="Collapsed"/>
            <TextBlock Text="ORD" Margin="0,0,4,0" VerticalAlignment="Center"/>
            <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ContentPresenter}, Path=ContentPresenter.ContentIndex}"
                       VerticalAlignment="Center" Margin="0,0,8,0"/>
            <ComboBox Width="100"
                      ItemsSource="{Binding DataContext.PatternIndexOptions, RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      SelectedItem="{Binding Mode=TwoWay}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- FCNT：三個 16 進位 digit（FCNT1 / FCNT2） -->
    <StackPanel Orientation="Horizontal" Margin="0,8,0,0">
      <TextBlock Text="FCNT1:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD2Options}" SelectedItem="{Binding FCNT1_D2, Mode=TwoWay}" ItemStringFormat="{}{0:X}" Margin="0,0,4,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD1Options}" SelectedItem="{Binding FCNT1_D1, Mode=TwoWay}" ItemStringFormat="{}{0:X}" Margin="0,0,4,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD0Options}" SelectedItem="{Binding FCNT1_D0, Mode=TwoWay}" ItemStringFormat="{}{0:X}" Margin="12,0,24,0"/>

      <TextBlock Text="FCNT2:" VerticalAlignment="Center" Margin="0,0,6,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD2Options}" SelectedItem="{Binding FCNT2_D2, Mode=TwoWay}" ItemStringFormat="{}{0:X}" Margin="0,0,4,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD1Options}" SelectedItem="{Binding FCNT2_D1, Mode=TwoWay}" ItemStringFormat="{}{0:X}" Margin="0,0,4,0"/>
      <ComboBox Width="55" ItemsSource="{Binding FcntD0Options}" SelectedItem="{Binding FCNT2_D0, Mode=TwoWay}" ItemStringFormat="{}{0:X}"/>
    </StackPanel>

    <Button Content="Set Auto-Run" Width="120" Height="30" Margin="0,12,0,0"
            Command="{Binding ApplyCommand}"/>
  </StackPanel>
</GroupBox>