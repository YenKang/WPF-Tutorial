<GroupBox Margin="12">
  <GroupBox.Header>
    <StackPanel Orientation="Horizontal">
      <TextBlock Text="Pattern En" FontWeight="Bold" Margin="0,0,8,0"/>
      <TextBlock Foreground="#2196F3">
        <TextBlock.Text>
          <MultiBinding StringFormat="— Selected: P{0} — {1}">
            <Binding Path="SelectedPattern.Index"/>
            <Binding Path="SelectedPattern.Name"/>
          </MultiBinding>
        </TextBlock.Text>
      </TextBlock>
    </StackPanel>
  </GroupBox.Header>

  <!-- 下面放你的 ListBox… -->
</GroupBox>