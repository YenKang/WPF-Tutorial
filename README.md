## Step 1：在 XAML 外面包一個 TabControl


```
<TabControl>

    <!-- Tab 1: 原本畫面 -->
    <TabItem Header="Main Setting">
        <!-- 這裡放你現在整個 Grid / Layout（完整搬進來） -->
    </TabItem>

    <!-- Tab 2: 新的 ASCII 頁面 -->
    <TabItem Header="ASCII">
        <!-- 等一下 Step 3 來填 -->
    </TabItem>

</TabControl>
```

## Step 2：ViewModel 新增一個「ASCII 列資料模型」

```
public sealed class RegAsciiSlotModel : ViewModelBase
{
    public string RegName { get; }

    private ImageOption _selectedImage;
    public ImageOption SelectedImage
    {
        get { return _selectedImage; }
        set
        {
            if (_selectedImage != value)
            {
                _selectedImage = value;
                RaisePropertyChanged(nameof(SelectedImage));
                // 之後如果要寫暫存器，可在這裡或外面做
            }
        }
    }

    public RegAsciiSlotModel(string regName)
    {
        RegName = regName;
    }
}
```

### step2-1: 在 IconToImageMapViewModel 裡加一個集合：


```
public ObservableCollection<RegAsciiSlotModel> AsciiSlots { get; }
    = new ObservableCollection<RegAsciiSlotModel>();

public void InitAsciiSlots()
{
    AsciiSlots.Clear();

    // 這兩個暫存器名稱你之後可以換成正式的
    AsciiSlots.Add(new RegAsciiSlotModel("OSD_ASCII_REG1"));
    AsciiSlots.Add(new RegAsciiSlotModel("OSD_ASCII_REG2"));
}
```

### step2-2: 在 ViewModel 建構子裡面最後呼叫一次：


```
public IconToImageMapViewModel()
{
    // ...你原本的初始化
    InitAsciiSlots();
}
```


## Step 3：ASCII Tab 的 XAML（兩欄：RegName + 選圖片按鈕）



```
<TabItem Header="ASCII">
    <Grid Margin="8">
        <ItemsControl ItemsSource="{Binding AsciiSlots}">
            <ItemsControl.ItemTemplate>
                <DataTemplate>
                    <Grid Margin="0,4">
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition Width="2*" />   <!-- Reg Name -->
                            <ColumnDefinition Width="*" />    <!-- 按鈕 -->
                        </Grid.ColumnDefinitions>

                        <!-- 左：暫存器名稱 -->
                        <TextBlock Grid.Column="0"
                                   Text="{Binding RegName}"
                                   VerticalAlignment="Center"/>

                        <!-- 右：選圖片按鈕 -->
                        <Button Grid.Column="1"
                                Margin="8,0,0,0"
                                Content="{Binding SelectedImage.Key, FallbackValue=選圖片}"
                                Click="OpenAsciiIconPicker_Click"/>
                    </Grid>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
    </Grid>
</TabItem>
```

## Step 4：OpenAsciiIconPicker_Click — 用「前 16 個 OSD ICON SELECT」當素材

```
private void OpenAsciiIconPicker_Click(object sender, RoutedEventArgs e)
{
    // 1) 取得目前這一行的 RegAsciiSlotModel
    var btn = (Button)sender;
    if (!(btn.DataContext is RegAsciiSlotModel slot))
        return;

    // 2) 拿到整個視窗的 ViewModel
    if (!(this.DataContext is IconToImageMapViewModel vm))
        return;

    // 3) 建立候選清單：來源 = OSD ICON SELECT 前 16 列
    var candidates = vm.IconSlots
        .Take(16)                                  // 前 16 個 OSD
        .Select(s => s.OsdSelectedImage)           // 每列選到的圖
        .Where(img => img != null)                 // 排除尚未選圖
        .ToList();

    if (candidates.Count == 0)
    {
        MessageBox.Show("前 16 個 OSD Icon 尚未選圖，無法選擇 ASCII 圖片。",
                        "提示", MessageBoxButton.OK, MessageBoxImage.Information);
        return;
    }

    // 4) 傳入目前已選的 key（可為 null）
    string preKey = slot.SelectedImage?.Key;

    // 5) 目前允許重複使用同一張圖，因此 usedKeys 先給空集合
    var usedKeys = new HashSet<string>();

    // 6) 打開你原本的 ImagePickerWindow
    var picker = new ImagePickerWindow(candidates, preKey, usedKeys)
    {
        Owner = this
    };

    if (picker.ShowDialog() == true && picker.Selected != null)
    {
        // 7) 寫回這一列暫存器要綁的圖
        slot.SelectedImage = picker.Selected;
        // 之後如果要同時寫暫存器，可在這裡呼叫 vm 的某個方法
    }
}
```
