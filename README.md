// IconToImageMapWindow.xaml.cs
private readonly List<FlashButton> _flashButtonList;

public IconToImageMapWindow(IEnumerable<dynamic> buttonsList)
{
    InitializeComponent();

    // 建立 FlashButtonList（你之前已經有）
    _flashButtonList = new List<FlashButton>();
    foreach (dynamic b in buttonsList)
    {
        var fb = new FlashButton
        {
            ImageFilePath    = b.BMPFilePath,
            FlashStartAddress= b.ICON_SRAM_ADDR,
            IconSramByteSize = b.ICON_SRAM_ByteSize
        };
        _flashButtonList.Add(fb);
    }

    // 取得 / 建立 VM
    var vm = this.DataContext as IconToImageMapViewModel;
    if (vm == null)
    {
        vm = new IconToImageMapViewModel();
        this.DataContext = vm;
    }

    if (vm.IconSlots.Count == 0)
        vm.InitIconSlots();

    // ✅ 用 FlashButton 來載入圖片 + 建立 _imageSizeMap
    vm.LoadImagesFromFlashButtons(_flashButtonList);

    // ⚠️ 等一下 Step 3 會在這裡再加一行：訂閱 SelectedImage 變更
    vm.HookSlotEvents();
}