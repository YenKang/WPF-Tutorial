<!-- ============================= -->
<!--  Frame Count (FCNT1 / FCNT2)  -->
<!-- ============================= -->

<StackPanel Margin="0,12,0,0">

  <!-- 區段標題 -->
  <TextBlock Text="Frame Count" 
             FontWeight="Bold" 
             Margin="0,0,0,6"/>

  <!-- ========== FCNT1 ========== -->
  <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
    <TextBlock Text="FCNT1:" Width="60" VerticalAlignment="Center"/>

    <!-- D2: 0~3 -->
    <ComboBox Width="60"
              ItemsSource="{Binding D2Options}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT1_D2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <!-- D1: 0~F -->
    <ComboBox Width="60" Margin="6,0,0,0"
              ItemsSource="{Binding NibbleOptions}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT1_D1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <!-- D0: 0~F -->
    <ComboBox Width="60" Margin="6,0,0,0"
              ItemsSource="{Binding NibbleOptions}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT1_D0, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <!-- 右側顯示十進位（例：0x03C → 60） -->
    <TextBlock Margin="12,0,0,0"
               VerticalAlignment="Center"
               Text="{Binding FCNT1}"/>
  </StackPanel>

  <!-- ========== FCNT2 ========== -->
  <StackPanel Orientation="Horizontal" Margin="0,0,0,6">
    <TextBlock Text="FCNT2:" Width="60" VerticalAlignment="Center"/>

    <ComboBox Width="60"
              ItemsSource="{Binding D2Options}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT2_D2, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <ComboBox Width="60" Margin="6,0,0,0"
              ItemsSource="{Binding NibbleOptions}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT2_D1, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <ComboBox Width="60" Margin="6,0,0,0"
              ItemsSource="{Binding NibbleOptions}"
              SelectedValuePath="."
              SelectedValue="{Binding FCNT2_D0, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}">
      <ComboBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding StringFormat={}{0:X}}"/>
        </DataTemplate>
      </ComboBox.ItemTemplate>
    </ComboBox>

    <TextBlock Margin="12,0,0,0"
               VerticalAlignment="Center"
               Text="{Binding FCNT2}"/>
  </StackPanel>

</StackPanel>