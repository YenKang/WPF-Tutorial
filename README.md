<!-- 建議給根元素命名，方便用 ElementName 綁回 VM -->
<UserControl x:Class="..."
             x:Name="Root"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:i="http://schemas.microsoft.com/xaml/behaviors">

    <Grid>
        <ItemsControl ItemsSource="{Binding FanBarIndexList}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <!-- 這一列的 DataContext == 當前 index (0..N-1) -->
                    <Rectangle Height="30"
                               Width="300"
                               Margin="25"
                               Cursor="Hand">
                        <Rectangle.Fill>
                            <!-- ✅ 原封不動保留你的 MultiConverter -->
                            <MultiBinding Converter="{StaticResource FanLevelToBrushMultiConverter}">
                                <!-- currentLevel（1-based） -->
                                <Binding Path="DataContext.LeftFanLevel"
                                         RelativeSource="{RelativeSource AncestorType=UserControl}" />
                                <!-- 這條的 index（0-based） -->
                                <Binding />
                            </MultiBinding>
                        </Rectangle.Fill>

                        <!-- ✅ 用 Behaviors 把滑鼠事件 → Command，不改任何視覺層 -->
                        <i:Interaction.Triggers>
                            <i:EventTrigger EventName="MouseLeftButtonDown">
                                <i:InvokeCommandAction
                                    Command="{Binding DataContext.SetLeftFanLevelCmd, ElementName=Root}"
                                    CommandParameter="{Binding}" />
                            </i:EventTrigger>
                        </i:Interaction.Triggers>
                    </Rectangle>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </Grid>
</UserControl>