<!-- 這段放在 ItemsControl.ItemTemplate 裡你的四欄 Grid -->
<!-- col2: D2 -->
<ComboBox Grid.Column="1" Width="90" Height="28" HorizontalAlignment="Center"
          ItemsSource="{StaticResource Range0to3}"
          SelectedItem="{Binding DataContext.D2, Mode=TwoWay,
                                 RelativeSource={RelativeSource AncestorType=Window}}"
          Background="White" Foreground="Black"/>

<!-- col3: D1 -->
<ComboBox Grid.Column="2" Width="90" Height="28" HorizontalAlignment="Center"
          ItemsSource="{StaticResource Range0toF}"
          SelectedItem="{Binding DataContext.D1, Mode=TwoWay,
                                 RelativeSource={RelativeSource AncestorType=Window}}"
          Background="White" Foreground="Black"/>

<!-- col4: D0 -->
<ComboBox Grid.Column="3" Width="90" Height="28" HorizontalAlignment="Center"
          ItemsSource="{StaticResource Range0toF}"
          SelectedItem="{Binding DataContext.D0, Mode=TwoWay,
                                 RelativeSource={RelativeSource AncestorType=Window}}"
          Background="White" Foreground="Black"/>