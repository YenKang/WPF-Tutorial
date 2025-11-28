## 1CspSlotModel.cs（最終版，無 clamp AlphaSel）

<GroupBox Header="CSP Config" Margin="8">
    <Grid>
        <Grid.ColumnDefinitions>
            <!-- CSP Register -->
            <ColumnDefinition Width="150"/>
            <!-- Value -->
            <ColumnDefinition Width="120"/>
            <!-- 間隔 -->
            <ColumnDefinition Width="20"/>
            <!-- UI Preview -->
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- ========== CSP_ALPHA_EN ========== -->
        <TextBlock Grid.Row="0" Grid.Column="0"
                   Text="CSP_ALPHA_EN" Margin="0,0,4,4"
                   VerticalAlignment="Center"/>

        <CheckBox Grid.Row="0" Grid.Column="1"
                  IsChecked="{Binding CspSlot.AlphaEnable, Mode=TwoWay}"
                  Content="Enable"
                  Margin="0,0,4,4"
                  VerticalAlignment="Center"/>

        <!-- ========== CSP_ALPHA_SEL ========== -->
        <TextBlock Grid.Row="1" Grid.Column="0"
                   Text="CSP_ALPHA_SEL[5:0]" Margin="0,0,4,4"
                   VerticalAlignment="Center"/>

        <!-- 這裡用 0~63，範圍照 datasheet；VM 本身不 clamp -->
        <xctk:IntegerUpDown Grid.Row="1" Grid.Column="1"
                            Minimum="0"
                            Maximum="63"
                            Value="{Binding CspSlot.AlphaSel, Mode=TwoWay}"
                            Margin="0,0,4,4"
                            VerticalContentAlignment="Center"/>

        <!-- ========== CSP_R ========== -->
        <TextBlock Grid.Row="2" Grid.Column="0"
                   Text="CSP_R[7:0]" Margin="0,0,4,4"
                   VerticalAlignment="Center"/>

        <xctk:IntegerUpDown Grid.Row="2" Grid.Column="1"
                            Minimum="0"
                            Maximum="255"
                            Value="{Binding CspSlot.R, Mode=TwoWay}"
                            Margin="0,0,4,4"
                            VerticalContentAlignment="Center"/>

        <!-- ========== CSP_G ========== -->
        <TextBlock Grid.Row="3" Grid.Column="0"
                   Text="CSP_G[7:0]" Margin="0,0,4,4"
                   VerticalAlignment="Center"/>

        <xctk:IntegerUpDown Grid.Row="3" Grid.Column="1"
                            Minimum="0"
                            Maximum="255"
                            Value="{Binding CspSlot.G, Mode=TwoWay}"
                            Margin="0,0,4,4"
                            VerticalContentAlignment="Center"/>

        <!-- ========== CSP_B ========== -->
        <TextBlock Grid.Row="4" Grid.Column="0"
                   Text="CSP_B[7:0]" Margin="0,0,0,0"
                   VerticalAlignment="Center"/>

        <xctk:IntegerUpDown Grid.Row="4" Grid.Column="1"
                            Minimum="0"
                            Maximum="255"
                            Value="{Binding CspSlot.B, Mode=TwoWay}"
                            Margin="0,0,4,0"
                            VerticalContentAlignment="Center"/>

        <!-- ========== 右側 Preview：跨 5 列 ========== -->
        <Border Grid.Row="0" Grid.RowSpan="5" Grid.Column="3"
                Margin="12,0,0,0"
                BorderBrush="DarkGreen"
                BorderThickness="2"
                SnapsToDevicePixels="True">
            <Rectangle Fill="{Binding CspSlot.PreviewBrush}"
                       Opacity="{Binding CspSlot.PreviewOpacity}"/>
        </Border>
    </Grid>
</GroupBox>

## 2
using System.Windows.Media;

namespace BistMode.Models   // 你如果 AsilSlotModel 放在別的 namespace，就跟著調整
{
    public sealed class CspSlotModel : ViewModelBase
    {
        // ===== Register Name（固定文字，給寫暫存器用） =====

        public string AlphaEnRegName  { get; }
        public string AlphaSelRegName { get; }
        public string RRegName        { get; }
        public string GRegName        { get; }
        public string BRegName        { get; }

        // ===== Raw 值：純粹存資料，不做 clamp =====

        private bool _alphaEnable;
        public bool AlphaEnable
        {
            get { return _alphaEnable; }
            set
            {
                if (_alphaEnable != value)
                {
                    _alphaEnable = value;
                    RaisePropertyChanged(nameof(AlphaEnable));
                    UpdatePreview();
                }
            }
        }

        // AlphaSel 範圍 datasheet 是 0~63，但這裡「不 clamp」，
        // 把檢查交給 UI 或外面處理。
        private int _alphaSel;
        public int AlphaSel
        {
            get { return _alphaSel; }
            set
            {
                if (_alphaSel != value)
                {
                    _alphaSel = value;
                    RaisePropertyChanged(nameof(AlphaSel));
                    UpdatePreview();
                }
            }
        }

        private int _r;
        public int R
        {
            get { return _r; }
            set
            {
                if (_r != value)
                {
                    _r = value;
                    RaisePropertyChanged(nameof(R));
                    UpdatePreview();
                }
            }
        }

        private int _g;
        public int G
        {
            get { return _g; }
            set
            {
                if (_g != value)
                {
                    _g = value;
                    RaisePropertyChanged(nameof(G));
                    UpdatePreview();
                }
            }
        }

        private int _b;
        public int B
        {
            get { return _b; }
            set
            {
                if (_b != value)
                {
                    _b = value;
                    RaisePropertyChanged(nameof(B));
                    UpdatePreview();
                }
            }
        }

        // ===== 給 UI 用的屬性 =====

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

        private double _previewOpacity = 1.0;
        public double PreviewOpacity
        {
            get { return _previewOpacity; }
            private set
            {
                if (_previewOpacity != value)
                {
                    _previewOpacity = value;
                    RaisePropertyChanged(nameof(PreviewOpacity));
                }
            }
        }

        // ===== 建構子 =====

        public CspSlotModel(
            string alphaEnRegName,
            string alphaSelRegName,
            string rRegName,
            string gRegName,
            string bRegName)
        {
            AlphaEnRegName  = alphaEnRegName;
            AlphaSelRegName = alphaSelRegName;
            RRegName        = rRegName;
            GRegName        = gRegName;
            BRegName        = bRegName;

            // 預設值可以依 datasheet/需求調整
            _alphaEnable = false;
            _alphaSel    = 0;     // 00h default
            _r           = 0xFF;  // datasheet default: FFh, 90h, 90h
            _g           = 0x90;
            _b           = 0x90;

            UpdatePreview();
        }

        // ===== UI Preview 計算 =====

        private void UpdatePreview()
        {
            // 1. 顏色：為了防止 UI 畫刷爆掉，這裡只對「輸出值」做 clamp，
            //    不改動 R/G/B 本身（raw 值保持原樣）。
            byte cr = (byte)System.Math.Max(0, System.Math.Min(255, _r));
            byte cg = (byte)System.Math.Max(0, System.Math.Min(255, _g));
            byte cb = (byte)System.Math.Max(0, System.Math.Min(255, _b));

            var color = Color.FromRgb(cr, cg, cb);
            var brush = new SolidColorBrush(color);
            brush.Freeze();
            PreviewBrush = brush;

            // 2. 透明度：AlphaEnable 啟用時才吃 AlphaSel。
            if (_alphaEnable)
            {
                // AlphaSel 0~63 → 0.0~1.0
                double alpha = _alphaSel / 63.0;

                // 同樣只 clamp UI，避免 Opacity > 1 or < 0
                if (alpha < 0.0) alpha = 0.0;
                if (alpha > 1.0) alpha = 1.0;

                PreviewOpacity = alpha;
            }
            else
            {
                // 沒啟用透明度權重 → 當作完全不透明
                PreviewOpacity = 1.0;
            }
        }

        // =====（選用）對外提供批次設定 API，方便外面載入 JSON / 讀暫存器 =====

        public void SetRawValues(bool alphaEn, int alphaSel, int r, int g, int b)
        {
            _alphaEnable = alphaEn;
            _alphaSel    = alphaSel;
            _r           = r;
            _g           = g;
            _b           = b;

            // 一次改完後，手動通知 PropertyChanged
            RaisePropertyChanged(nameof(AlphaEnable));
            RaisePropertyChanged(nameof(AlphaSel));
            RaisePropertyChanged(nameof(R));
            RaisePropertyChanged(nameof(G));
            RaisePropertyChanged(nameof(B));

            UpdatePreview();
        }
    }
}

##. 3
public sealed class BistModeViewModel : ViewModelBase
{
    public CspSlotModel CspSlot { get; }

    public BistModeViewModel()
    {
        CspSlot = new CspSlotModel(
            "CSP_ALPHA_EN",
            "CSP_ALPHA_SEL",
            "CSP_R",
            "CSP_G",
            "CSP_B");
    }
}


