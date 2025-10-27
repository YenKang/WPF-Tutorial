<!-- 記得在頂部 xmlns 加上：xmlns:local="clr-namespace:YourAppNamespace" -->

<StackPanel Orientation="Horizontal" Margin="0,4,0,0">
  <TextBox Width="80">
    <TextBox.Text>
      <Binding Path="ChessBoardVM.HRes"
               Mode="TwoWay"
               UpdateSourceTrigger="PropertyChanged">
        <Binding.ValidationRules>
          <local:IntRangeRule Min="720" Max="6720"/>
        </Binding.ValidationRules>
      </Binding>
    </TextBox.Text>
  </TextBox>

  <TextBlock Text=" × " Margin="6,0"/>

  <TextBox Width="80">
    <TextBox.Text>
      <Binding Path="ChessBoardVM.VRes"
               Mode="TwoWay"
               UpdateSourceTrigger="PropertyChanged">
        <Binding.ValidationRules>
          <local:IntRangeRule Min="136" Max="2560"/>
        </Binding.ValidationRules>
      </Binding>
    </TextBox.Text>
  </TextBox>
</StackPanel>