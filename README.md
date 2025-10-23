<ComboBox Width="60" Margin="0,0,6,0"
          ItemsSource="{Binding BWLoopVM.D2Options}"
          SelectedItem="{Binding BWLoopVM.FCNT2_D2, Mode=TwoWay}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding StringFormat={}{0:X}}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>

<ComboBox Width="60" Margin="0,0,6,0"
          ItemsSource="{Binding BWLoopVM.D1Options}"
          SelectedItem="{Binding BWLoopVM.FCNT2_D1, Mode=TwoWay}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding StringFormat={}{0:X}}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>

<ComboBox Width="60" Margin="0,0,12,0"
          ItemsSource="{Binding BWLoopVM.D0Options}"
          SelectedItem="{Binding BWLoopVM.FCNT2_D0, Mode=TwoWay}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding StringFormat={}{0:X}}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>