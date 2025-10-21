xmlns:vm="clr-namespace:BistMode.ViewModels"

<GroupBox Header="Auto-Run Config"
          Visibility="{Binding IsAutoRunConfigVisible,
                               Converter={StaticResource BoolToVisibilityConverter}}"
          Margin="0,12,0,0">
  <!-- 整個群組直接綁 AutoRunVM 當 DataContext -->
  <StackPanel Margin="12" DataContext="{Binding AutoRunVM}">

    <!-- Pattern Total -->
    <StackPanel Orientation="Horizontal" Margin="0,0,0,8">
      <TextBlock Text="Pattern Total:" VerticalAlignment="Center" Margin="0,0,8,0"/>
      <ComboBox Width="80"
                ItemsSource="{Binding TotalOptions}"
                SelectedItem="{Binding Total, Mode=TwoWay}"/>
    </StackPanel>

    <!-- ORD0..ORD(Total-1) -->
    <ItemsControl ItemsSource="{Binding Orders}" AlternationCount="200">
      <ItemsControl.ItemTemplate>
        <DataTemplate>
          <StackPanel Orientation="Horizontal" Margin="0,2">
            <!-- 用 AlternationIndex 當作顯示的序號 -->
            <TextBlock Width="70" VerticalAlignment="Center"
                       Text="{Binding (ItemsControl.AlternationIndex),
                                      RelativeSource={RelativeSource Mode=FindAncestor, AncestorType=ContentPresenter},
                                      StringFormat=ORD{0:D2}:}"/>
            <ComboBox Width="160"
                      ItemsSource="{Binding DataContext.PatternIndexOptions,
                                            RelativeSource={RelativeSource AncestorType=GroupBox}}"
                      SelectedItem="{Binding Mode=TwoWay}"/>
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>

    <!-- FCNT1 / FCNT2 三位 16 進位 digit -->
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