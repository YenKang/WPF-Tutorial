<!-- 可重用：底線樣式（只畫下邊線） -->
<Style x:Key="BottomLineBase" TargetType="Border">
  <Setter Property="BorderThickness" Value="0,0,0,2"/>
  <Setter Property="Margin" Value="2"/>
</Style>

<Style x:Key="BottomLineRed" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Red"/>
</Style>
<Style x:Key="BottomLineGreen" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Green"/>
</Style>
<Style x:Key="BottomLineBlue" TargetType="Border" BasedOn="{StaticResource BottomLineBase}">
  <Setter Property="BorderBrush" Value="Blue"/>
</Style>