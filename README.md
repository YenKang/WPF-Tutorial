public partial class IconToImageMapWindow : Window
{
    private List<FlashButton> _flashButtonList;

    public IconToImageMapWindow(IEnumerable<dynamic> ButtonsList)
    {
        InitializeComponent();

        // 1️⃣ 先把 ButtonsList 轉成 FlashButtonList
        _flashButtonList = new List<FlashButton>();

        foreach (dynamic b in ButtonsList)
        {
            _flashButtonList.Add(new FlashButton
            {
                ImageFilePath     = b.BMPFilePath,
                FlashStartAddress = b.ICON_SRAM_ADDR,
                IconSramByteSize  = b.ICON_SRAM_ByteSize
            });
        }

        // 2️⃣ 取得 / 建立 ViewModel，並設定 DataContext
        IconToImageMapViewModel vm;
        if (this.DataContext is IconToImageMapViewModel existVm)
        {
            vm = existVm;
        }
        else
        {
            vm = new IconToImageMapViewModel();
            this.DataContext = vm;
        }

        // 3️⃣ 初始化 IconSlots（如果你還是需要 30 列）
        if (vm.IconSlots.Count == 0)
        {
            vm.InitIconSlots();
        }

        // ❌ 之前是用固定資料夾載入假圖，現在先不要：
        // if (vm.Images.Count == 0)
        // {
        //     vm.LoadImagesFromFolder(@"Assets\TestIcons");
        // }

        // ✅ 4️⃣ 改成用 FlashButtonList 當圖片牆素材來源
        if (_flashButtonList.Count > 0)
        {
            vm.LoadImagesFromFlashButtons(_flashButtonList);
        }
    }
}