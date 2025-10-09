<GroupBox Margin="12">
  <GroupBox.Header>
    <Border Background="#E0E0E0" Padding="6" CornerRadius="2">
      <StackPanel Orientation="Horizontal">
        <TextBlock Text="Pattern En" FontWeight="Bold" Foreground="#000" />
        <TextBlock Margin="8,0,0,0" Foreground="#333">
          <TextBlock.Text>
            <MultiBinding StringFormat="[P{0}] - {1}">
              <Binding Path="SelectedPattern.Index" TargetNullValue="-" />
              <Binding Path="SelectedPattern.Name"  TargetNullValue="(none)" />
            </MultiBinding>
          </TextBlock.Text>
        </TextBlock>
      </StackPanel>
    </Border>
  </GroupBox.Header>

  <!-- 這裡放你的 ListBox ... -->
</GroupBox>