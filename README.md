
## 1. ASIL 用的候選項目 class

public sealed class AsilIconCandidate
{
    public int OsdIndex { get; private set; }      // OSD # (1~30)
    public string DisplayName { get; private set; } // 圖片顯示名稱，例如「2右轉燈」
    public string ImageKey { get; private set; }    // 對應到 Image 的 key

    public string ComboDisplay   // 給 ComboBox 顯示用
    {
        get { return string.Format("OSD #{0} - {1}", OsdIndex, DisplayName); }
    }

    public AsilIconCandidate(int osdIndex, string displayName, string imageKey)
    {
        OsdIndex    = osdIndex;
        DisplayName = displayName;
        ImageKey    = imageKey;
    }
}

## 2. ViewModel 裡的清單 & 重建邏輯

假設你已經有 ObservableCollection<OSDICButton> OSDICButtonList：

```
public ObservableCollection<AsilIconCandidate> AsilIconCandidates
    = new ObservableCollection<AsilIconCandidate>();

public void RebuildAsilIconCandidates()
{
    AsilIconCandidates.Clear();

    foreach (var btn in OSDICButtonList)
    {
        if (btn.SelectedImage == null)
            continue;

        // 這裡假設 btn 有 OsdIndex、SelectedImage.DisplayName、SelectedImage.ImageKey
        AsilIconCandidates.Add(
            new AsilIconCandidate(
                btn.OsdIndex,                     // OSD #3
                btn.SelectedImage.DisplayName,    // "2右轉燈"
                btn.SelectedImage.ImageKey        // "turn_right_2" 類似這種
            )
        );
    }
}
```


這個 RebuildAsilIconCandidates() 你可以在：
	•	Main Setting 改完 OSD ICON SELECT 之後呼叫一次
	•	或是在 OSDICButtonList Item 改變時呼叫


## ViewModel

```
private AsilIconCandidate _selectedGateOsd;
public AsilIconCandidate SelectedGateOsd
{
    get { return _selectedGateOsd; }
    set
    {
        if (_selectedGateOsd != value)
        {
            _selectedGateOsd = value;
            RaisePropertyChanged("SelectedGateOsd");

            // 實際要寫進暫存器的 Gate_OSD_SEL 值
            GateOsdSelValue = (value != null) ? value.OsdIndex : 0;
            RaisePropertyChanged("GateOsdSelValue");
        }
    }
}

// 這個 int 之後 Apply/WriteReg 時，就拿來寫 Gate_OSD_SEL 那顆暫存器
public int GateOsdSelValue { get; private set; }
```

```
<ComboBox
    ItemsSource="{Binding AsilIconCandidates}"
    SelectedItem="{Binding SelectedGateOsd, Mode=TwoWay}"
    DisplayMemberPath="ComboDisplay"
    Width="180"
    />
```
