public IconToImageMapWindow(IEnumerable<dynamic> ButtonsList)
{
    InitializeComponent();

    FlashButtonList = new List<FlashButton>();

    int index = 0;
    foreach (dynamic b in ButtonsList)
    {
        // 建立一筆 FlashButton 記錄
        var fb = new FlashButton
        {
            ImageFilePath      = b.BMPFilePath,        // 圖片實際路徑
            FlashStartAddress  = b.ICON_SRAM_ADDR,     // 計算好的 Start Address
            IconSramByteSize   = b.ICON_SRAM_ByteSize  // 計算好的 ByteSize
        };

        FlashButtonList.Add(fb);
        index++;
    }
}

=====

foreach (var fb in FlashButtonList)
{
    Debug.WriteLine(
        $"FlashButton => Path={fb.ImageFilePath}, Start=0x{fb.FlashStartAddress:X}, Size={fb.IconSramByteSize}");
}