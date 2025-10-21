<ComboBox Width="160"
          ItemsSource="{Binding DataContext.AutoRunVM.PatternOptions,
                                RelativeSource={RelativeSource AncestorType=GroupBox}}"
          DisplayMemberPath="Display"
          SelectedValuePath="Index"
          SelectedValue="{Binding SelectedIndex, Mode=TwoWay}" />

＝＝＝＝＝＝

public ObservableCollection<PatternOption> PatternOptions { get; }
    = new ObservableCollection<PatternOption>();