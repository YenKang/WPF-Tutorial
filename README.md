<StackPanel Margin="20">
    <TextBlock Text="I am visible when SelectedPattern is NOT null"
               Foreground="Green"
               Visibility="{Binding SelectedPattern, Converter={StaticResource NullToVisibility}}"/>

    <TextBlock Text="I am visible when SelectedPattern is NULL"
               Foreground="Red"
               Visibility="{Binding SelectedPattern, Converter={StaticResource NullToInvertedVisibility}}"/>
</StackPanel>