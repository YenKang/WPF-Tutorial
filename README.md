

## 1.Model: CspSlotModel（只是資料）

```
public sealed class CspSlotModel : ViewModelBase
{
    public string RegName { get; private set; }

    public int Min  { get; private set; }
    public int Max  { get; private set; }

    private int _value;
    public int Value
    {
        get { return _value; }
        set
        {
            if (_value != value)
            {
                _value = value;
                RaisePropertyChanged(nameof(Value));
            }
        }
    }

    public CspSlotModel(string regName, int min, int max, int defaultValue)
    {
        RegName = regName;
        Min     = min;
        Max     = max;
        _value  = defaultValue;
    }
}
```

## 1-2 : CspColorPreviewModel.cs

```
using System;
using System.Windows.Media;
using Utility.MVVM;

public sealed class CspColorPreviewModel : ViewModelBase
{
    private int _alphaEn;   // CSP_ALPHA_EN : 0 / 1
    private int _alphaSel;  // CSP_ALPHA_SEL : 0~63
    private int _r;         // CSP_R : 0~255
    private int _g;         // CSP_G : 0~255
    private int _b;         // CSP_B : 0~255

    public int AlphaEn
    {
        get { return _alphaEn; }
        set
        {
            if (_alphaEn != value)
            {
                _alphaEn = value;
                RaisePropertyChanged(nameof(AlphaEn));
                RaisePropertyChanged(nameof(ColorBrush));
            }
        }
    }

    public int AlphaSel
    {
        get { return _alphaSel; }
        set
        {
            if (_alphaSel != value)
            {
                _alphaSel = value;
                RaisePropertyChanged(nameof(AlphaSel));
                RaisePropertyChanged(nameof(ColorBrush));
            }
        }
    }

    public int R
    {
        get { return _r; }
        set
        {
            if (_r != value)
            {
                _r = value;
                RaisePropertyChanged(nameof(R));
                RaisePropertyChanged(nameof(ColorBrush));
            }
        }
    }

    public int G
    {
        get { return _g; }
        set
        {
            if (_g != value)
            {
                _g = value;
                RaisePropertyChanged(nameof(G));
                RaisePropertyChanged(nameof(ColorBrush));
            }
        }
    }

    public int B
    {
        get { return _b; }
        set
        {
            if (_b != value)
            {
                _b = value;
                RaisePropertyChanged(nameof(B));
                RaisePropertyChanged(nameof(ColorBrush));
            }
        }
    }

    // 照 datasheet：
    // Alpha = AlphaSel / 63.0 (En=0 → Alpha=1.0)
    // OutR = R * Alpha, OutG = G * Alpha, OutB = B * Alpha
    public SolidColorBrush ColorBrush
    {
        get
        {
            int alphaEn  = _alphaEn;
            int alphaSel = _alphaSel;
            int r        = _r;
            int g        = _g;
            int b        = _b;

            if (alphaSel < 0)  alphaSel = 0;
            if (alphaSel > 63) alphaSel = 63;
            if (r < 0) r = 0; if (r > 255) r = 255;
            if (g < 0) g = 0; if (g > 255) g = 255;
            if (b < 0) b = 0; if (b > 255) b = 255;

            double alpha = (alphaEn != 0)
                ? (alphaSel / 63.0)
                : 1.0;

            int outR = (int)Math.Round(r * alpha);
            int outG = (int)Math.Round(g * alpha);
            int outB = (int)Math.Round(b * alpha);

            if (outR < 0) outR = 0; if (outR > 255) outR = 255;
            if (outG < 0) outG = 0; if (outG > 255) outG = 255;
            if (outB < 0) outB = 0; if (outB > 255) outB = 255;

            return new SolidColorBrush(Color.FromRgb(
                (byte)outR,
                (byte)outG,
                (byte)outB
            ));
        }
    }
}
```

## 2️⃣ ViewModel：集中處理「色塊要長什麼樣」:IconToImageMapViewModel


### 2-1: 宣告欄位

```
public sealed class IconToImageMapViewModel : ViewModelBase
{
    // 你原本就有的成員 …
    // ...

    // === CSP 專用欄位 ===
    public ObservableCollection<CspSlotModel> CspSlots { get; }
        = new ObservableCollection<CspSlotModel>();

    public CspColorPreviewModel CspPreview { get; }
        = new CspColorPreviewModel();

    private CspSlotModel _cspAlphaEn;
    private CspSlotModel _cspAlphaSel;
    private CspSlotModel _cspR;
    private CspSlotModel _cspG;
    private CspSlotModel _cspB;

    // ...
}
```

### 2-2: 建構子初始化

```
public IconToImageMapViewModel()
{
    // 你原本的初始化流程 …
    // InitImageList();
    // InitOsdSlots();
    // ...

    InitCspSlots();   // ✅ 新增這行
}
```

### 2-3. 把 InitCspSlots() 實作進去

```
private void InitCspSlots()
{
    CspSlots.Clear();

    _cspAlphaEn = NewCspSlot("CSP_ALPHA_EN",  0, 1,    0);
    _cspAlphaSel= NewCspSlot("CSP_ALPHA_SEL", 0, 0x3F, 0);
    _cspR       = NewCspSlot("CSP_R",         0, 255,  255);
    _cspG       = NewCspSlot("CSP_G",         0, 255,  0);
    _cspB       = NewCspSlot("CSP_B",         0, 255,  0);

    // 初始化 preview （讓畫面一開始就有顏色）
    CspPreview.AlphaEn  = _cspAlphaEn.Value;
    CspPreview.AlphaSel = _cspAlphaSel.Value;
    CspPreview.R        = _cspR.Value;
    CspPreview.G        = _cspG.Value;
    CspPreview.B        = _cspB.Value;
}

private CspSlotModel NewCspSlot(string regName, int min, int max, int defaultValue)
{
    var slot = new CspSlotModel(regName, min, max, defaultValue);
    slot.PropertyChanged += OnCspSlotValueChanged;
    CspSlots.Add(slot);
    return slot;
}

private void OnCspSlotValueChanged(object sender, PropertyChangedEventArgs e)
{
    if (e.PropertyName != nameof(CspSlotModel.Value))
        return;

    // 任何一個 slot Value 改變 → 更新 preview 的參數
    CspPreview.AlphaEn  = _cspAlphaEn.Value;
    CspPreview.AlphaSel = _cspAlphaSel.Value;
    CspPreview.R        = _cspR.Value;
    CspPreview.G        = _cspG.Value;
    CspPreview.B        = _cspB.Value;
}
```


## 3️⃣ XAML：CSP Tab XAML：改成綁 IconToImageMapViewModel 的屬性

```
<TabItem Header="CSP">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Header -->
        <Grid Grid.Row="0" Margin="0,0,0,6">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="2.2*"/>
                <ColumnDefinition Width="1*"/>
                <ColumnDefinition Width="2.2*"/>
            </Grid.ColumnDefinitions>

            <TextBlock Grid.Column="0"
                       Text="CSP Register"
                       FontWeight="Bold"
                       FontSize="14"
                       Margin="4,0"/>

            <TextBlock Grid.Column="1"
                       Text="Value"
                       FontWeight="Bold"
                       FontSize="14"
                       HorizontalAlignment="Center"
                       Margin="4,0"/>

            <TextBlock Grid.Column="2"
                       Text="Preview"
                       FontWeight="Bold"
                       FontSize="14"
                       Margin="4,0"/>
        </Grid>

        <!-- 資料列 -->
        <ScrollViewer Grid.Row="1"
                      VerticalScrollBarVisibility="Auto">
            <ItemsControl ItemsSource="{Binding CspSlots}">
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Grid Margin="0,3">
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="2.2*"/>
                                <ColumnDefinition Width="1*"/>
                                <ColumnDefinition Width="2.2*"/>
                            </Grid.ColumnDefinitions>

                            <!-- 第一欄：RegName -->
                            <TextBlock Grid.Column="0"
                                       Text="{Binding RegName}"
                                       Margin="4,0"
                                       VerticalAlignment="Center"/>

                            <!-- 第二欄：Value（直接綁 slot.Value） -->
                            <TextBox Grid.Column="1"
                                     Text="{Binding Value, UpdateSourceTrigger=PropertyChanged}"
                                     HorizontalContentAlignment="Center"
                                     VerticalAlignment="Center"
                                     Margin="4,0"
                                     Height="24"/>

                            <!-- 第三欄：Preview 色塊（綁整個 IconToImageMapViewModel 的 CspPreview.ColorBrush） -->
                            <Border Grid.Column="2"
                                    Width="80"
                                    Height="60"
                                    Margin="4,0"
                                    BorderBrush="Black"
                                    BorderThickness="1"
                                    Background="{Binding DataContext.CspPreview.ColorBrush,
                                                         RelativeSource={RelativeSource AncestorType=TabItem}}"/>
                        </Grid>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </ScrollViewer>
    </Grid>
</TabItem>
```

