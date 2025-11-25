
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


＝＝＝＝＝＝＝＝＝＝＝＝


```
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Windows;
using System.Windows.Controls;

namespace YourNamespace
{
    public partial class ImagePickerWindow : Window
    {
        public ObservableCollection<ImageOption> Images { get; private set; }

        // 最後選擇的圖片
        public ImageOption SelectedImage { get; private set; }

        /// <summary>
        /// ✅ 給 ASIL / ASCII 用：外部指定一組候選圖片 + 目前選中的圖片
        /// </summary>
        public ImagePickerWindow(IEnumerable<ImageOption> sourceImages, ImageOption current)
        {
            InitializeComponent();

            if (sourceImages == null)
                throw new ArgumentNullException("sourceImages");

            Images = new ObservableCollection<ImageOption>(sourceImages);
            SelectedImage = current;

            DataContext = this;
        }

        /// <summary>
        /// ✅ 保留舊用法：如果你其他地方還是用「全部圖片」版本，可以在這裡加載。
        /// 若不需要，可以移除這個建構子。
        /// </summary>
        public ImagePickerWindow()
            : this(LoadAllImagesFromFlash(), null)
        {
        }

        /// <summary>
        /// 範例：舊的載入全部圖片的方法，若你已有實作，直接用你自己的即可。
        /// </summary>
        private static IList<ImageOption> LoadAllImagesFromFlash()
        {
            // TODO: 替換成你原本的「載入所有 icon 圖片」邏輯
            return new List<ImageOption>();
        }

        /// <summary>
        /// 點某一張圖的按鈕
        /// </summary>
        private void ImageButton_Click(object sender, RoutedEventArgs e)
        {
            var button = sender as Button;
            if (button == null)
                return;

            var img = button.DataContext as ImageOption;
            if (img == null)
                return;

            SelectedImage = img;
            DialogResult = true;
            Close();
        }

        /// <summary>
        /// 取消
        /// </summary>
        private void CancelButton_Click(object sender, RoutedEventArgs e)
        {
            DialogResult = false;
            Close();
        }
    }
}
```

using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Windows;
using System.Windows.Controls;

namespace YourNamespace
{
    public partial class AsciiIconMapWindow : Window
    {
        // 🟦 你原本的 OSD ICON SELECT 對應清單
        private readonly ObservableCollection<OSDICButton> _osdIcButtonList;

        // 🟨 ASIL / ASCII slot 列表（你原本就有）
        public ObservableCollection<RegAsciiSlotModel> AsilSlots { get; private set; }

        public AsciiIconMapWindow(ObservableCollection<OSDICButton> osdIcButtonList)
        {
            InitializeComponent();

            _osdIcButtonList = osdIcButtonList ?? throw new ArgumentNullException("osdIcButtonList");
            AsilSlots = new ObservableCollection<RegAsciiSlotModel>();

            // TODO: 初始化 AsilSlots，依你原本的 RegName 需求
            // 例如：
            // AsilSlots.Add(new RegAsciiSlotModel("REG_OSD_ASCI_0"));
            // AsilSlots.Add(new RegAsciiSlotModel("REG_OSD_ASCI_1"));
            // ...

            DataContext = this;
        }

        /// <summary>
        /// ✅ 核心：從 OSD ICON SELECT 抓「已選的圖」，依 OSD# 排序，取前 16 張
        /// </summary>
        private List<ImageOption> BuildAsilImageSource()
        {
            // 這邊假設 OSDICButton 有：
            //   int OsdIndex;
            //   ImageOption SelectedImage;
            // 若命名不同，請自己對應修改
            return _osdIcButtonList
                .Where(x => x.SelectedImage != null)
                .OrderBy(x => x.OsdIndex)
                .Select(x => x.SelectedImage)
                .Take(16)
                .ToList();
        }

        /// <summary>
        /// ✅ 在 ASIL/ASCII 頁面，點一格選圖時呼叫這個
        /// </summary>
        private void OpenAsilIconPicker(RegAsciiSlotModel slot)
        {
            if (slot == null)
                return;

            var asilImages = BuildAsilImageSource();
            if (asilImages.Count == 0)
            {
                MessageBox.Show("目前沒有任何 OSD ICON SELECT 已選圖可用。", "提示",
                    MessageBoxButton.OK, MessageBoxImage.Information);
                return;
            }

            var picker = new ImagePickerWindow(asilImages, slot.SelectedImage);
            picker.Owner = this;

            if (picker.ShowDialog() == true)
            {
                // 使用者選了一張圖
                slot.SelectedImage = picker.SelectedImage;

                // 如果這裡要順便寫暫存器，可以在這裡呼叫你的 RegWrite 相關函式
            }
        }

        /// <summary>
        /// ✅ XAML Button / Grid 的 Click 事件，綁到這裡
        /// </summary>
        private void AsilIconCellButton_Click(object sender, RoutedEventArgs e)
        {
            var btn = sender as Button;
            if (btn == null)
                return;

            var slot = btn.DataContext as RegAsciiSlotModel;
            if (slot == null)
                return;

            OpenAsilIconPicker(slot);
        }
    }
}
