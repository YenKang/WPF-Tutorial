## 1.model


public sealed class ColorConfigModel
{
    public byte R { get; set; }
    public byte G { get; set; }
    public byte B { get; set; }
}

## 2. viewmodel

using System.Windows.Media;

public sealed class ColorConfigVM : ViewModelBase
{
    private readonly ColorConfigModel _model;

    public ColorConfigVM(ColorConfigModel model)
    {
        _model = model ?? new ColorConfigModel { R = 255, G = 0, B = 0 }; // default: 紅色
        UpdatePreviewBrush();
    }

    public byte R
    {
        get { return _model.R; }
        set
        {
            if (_model.R != value)
            {
                _model.R = value;
                RaisePropertyChanged(nameof(R));
                UpdatePreviewBrush();
            }
        }
    }

    public byte G
    {
        get { return _model.G; }
        set
        {
            if (_model.G != value)
            {
                _model.G = value;
                RaisePropertyChanged(nameof(G));
                UpdatePreviewBrush();
            }
        }
    }

    public byte B
    {
        get { return _model.B; }
        set
        {
            if (_model.B != value)
            {
                _model.B = value;
                RaisePropertyChanged(nameof(B));
                UpdatePreviewBrush();
            }
        }
    }

    private SolidColorBrush _previewBrush;
    public SolidColorBrush PreviewBrush
    {
        get { return _previewBrush; }
        private set
        {
            if (_previewBrush != value)
            {
                _previewBrush = value;
                RaisePropertyChanged(nameof(PreviewBrush));
            }
        }
    }

    private void UpdatePreviewBrush()
    {
        // 每次輸入改變，就重建一顆新的 SolidColorBrush
        var color = Color.FromRgb(_model.R, _model.G, _model.B);
        var brush = new SolidColorBrush(color);
        brush.Freeze();                 // 可寫可不寫，寫了效能好一點
        PreviewBrush = brush;
    }
}

## 3.osd vm

public sealed class BistModeViewModel : ViewModelBase
{
    public ColorConfigVM ColorConfig { get; }

    public BistModeViewModel()
    {
        ColorConfig = new ColorConfigVM(new ColorConfigModel());
    }
}

## 4.xaml

<GroupBox Header="Color Config">
    <Grid Margin="8">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="80"/>
            <ColumnDefinition Width="20"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- R -->
        <TextBlock Grid.Row="0" Grid.Column="0" Text="R" VerticalAlignment="Center" Margin="0,0,4,4"/>
        <TextBox  Grid.Row="0" Grid.Column="1" Margin="0,0,4,4"
                  Text="{Binding ColorConfig.R,
                                 Mode=TwoWay,
                                 UpdateSourceTrigger=PropertyChanged}"/>

        <!-- G -->
        <TextBlock Grid.Row="1" Grid.Column="0" Text="G" VerticalAlignment="Center" Margin="0,0,4,4"/>
        <TextBox  Grid.Row="1" Grid.Column="1" Margin="0,0,4,4"
                  Text="{Binding ColorConfig.G,
                                 Mode=TwoWay,
                                 UpdateSourceTrigger=PropertyChanged}"/>

        <!-- B -->
        <TextBlock Grid.Row="2" Grid.Column="0" Text="B" VerticalAlignment="Center" Margin="0,0,4,0"/>
        <TextBox  Grid.Row="2" Grid.Column="1" Margin="0,0,4,0"
                  Text="{Binding ColorConfig.B,
                                 Mode=TwoWay,
                                 UpdateSourceTrigger=PropertyChanged}"/>

        <!-- 右邊色塊預覽 -->
        <Border Grid.Row="0" Grid.RowSpan="3" Grid.Column="3"
                Margin="8,0,0,0"
                BorderBrush="Black"
                BorderThickness="1">
            <Rectangle Fill="{Binding ColorConfig.PreviewBrush}"/>
        </Border>
    </Grid>
</GroupBox>
